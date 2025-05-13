# Chapter 2: FilePath

In [Chapter 1: File Endpoint & F2 Functions](01_file_endpoint___f2_functions_.md), we saw how to ask the file service to do things like list files (`FileListQuery`) or upload a new one (`FileUploadCommand`). You might have noticed that in those examples, we needed to specify *exactly which* file we were talking about or *where* a new file should go. How do we tell the system the precise location? That's where the `FilePath` comes in!

## What's a FilePath? The Address for Your Files

Imagine you want to send a letter. You can't just write "send this to John"; you need a full address: Street, City, Zip Code, Country. Similarly, in `connect-fs`, when you want to work with a file (get it, delete it, upload it), you need to provide its unique "address".

The `FilePath` is the standard way `connect-fs` identifies and locates every single file. It's like a structured postal address for your digital files stored in the system.

Think about organizing photos on your computer. You might create folders like `Photos / Vacations / Italy / Rome / colosseum.jpg`. `FilePath` provides a similar, but more structured, way to organize files within the storage system (like an [S3 Service](04_s3_service_.md) bucket).

## The Parts of the Address: Anatomy of a FilePath

A `FilePath` isn't just a single long name; it's broken down into four specific parts:

1.  **`objectType`**: What *kind* of thing does this file belong to? Is it related to a `project`, a `user`, an `organization`, a `product`? This helps group files logically.
    *   *Analogy:* The City in a postal address.
    *   *Example:* `"project"`

2.  **`objectId`**: Which specific instance of that type does the file belong to? If the `objectType` is `"project"`, the `objectId` would be the unique ID of that particular project.
    *   *Analogy:* The Street Name and House Number.
    *   *Example:* `"project-123-abc"`

3.  **`directory`**: Within that specific object, is the file in a particular category or subfolder? Maybe `images`, `documents`, `public`, `private`, `thumbnails`?
    *   *Analogy:* An Apartment Number or P.O. Box.
    *   *Example:* `"images"`

4.  **`name`**: Finally, what is the actual name of the file, including its extension?
    *   *Analogy:* The Recipient's Name on the envelope.
    *   *Example:* `"logo.png"`

Putting it all together, a `FilePath` uniquely points to one specific file related to one specific object.

## How to Use FilePath

Whenever you interact with the [File Endpoint & F2 Functions](01_file_endpoint___f2_functions_.md), you'll almost always need to provide a `FilePath`.

**Creating a FilePath:**

In the code, a `FilePath` is represented by a simple data structure (a `data class` in Kotlin).

```kotlin
// From fs-s2/file/fs-file-domain/src/commonMain/kotlin/io/komune/fs/s2/file/domain/model/FilePath.kt

@Serializable // Means this can be easily converted to text (like JSON)
data class FilePath(
    val objectType: String, // e.g., "project"
    val objectId: String,   // e.g., "project-123-abc"
    val directory: String,  // e.g., "images"
    val name: String        // e.g., "logo.png"
) {
    // ... helper functions ...
}
```

*   `@Serializable`: A label indicating this object can be sent over the network or saved easily.
*   `data class`: A concise way in Kotlin to define a class whose main purpose is to hold data.
*   The four fields directly correspond to the parts we just discussed.

**Example: Specifying a FilePath for Upload**

Remember the `FileUploadCommand` from Chapter 1? It requires a `FilePath` to know where to store the new file.

```kotlin
// Input for the fileUpload function
val uploadCommand = FileUploadCommand(
    path = FilePath( // Here's our FilePath!
        objectType = "project",
        objectId = "project-123-abc",
        directory = "images",
        name = "new-banner.jpg"
    ),
    metadata = mapOf("uploadedBy" to "Bob"),
    vectorize = false
)
// ... plus the actual file data ...
```

Here, we're telling the system: "Upload the file data I'm sending you, and store it as `new-banner.jpg` inside the `images` directory belonging to the `project` with ID `project-123-abc`."

**Example: Getting a FilePath in a Response**

When a file is uploaded successfully, the `FileUploadedEvent` includes the `FilePath` where the file was stored.

```kotlin
// Output from a successful fileUpload
val uploadResponse = FileUploadedEvent(
    id = "file-id-789",
    path = FilePath( // The FilePath is included here
        objectType = "project",
        objectId = "project-123-abc",
        directory = "images",
        name = "new-banner.jpg"
    ),
    url = "https://...",
    hash = "...",
    metadata = mapOf("uploadedBy" to "Bob", "id" to "file-id-789"),
    time = 1678890000000
)
```

This confirms exactly where the file ended up, using the same structured `FilePath`.

## Under the Hood: From FilePath to Storage Key

So, how does this structured `FilePath` actually translate into something the underlying storage system (like an [S3 Service](04_s3_service_.md) bucket) understands?

It's simple: the four parts are joined together with forward slashes (`/`) to create a single string path.

`objectType` / `objectId` / `directory` / `name`

Using our example:
`project/project-123-abc/images/new-banner.jpg`

This combined string acts as the unique key or filename within the storage bucket.

**Code Snippet: Creating the String Path**

The `FilePath` class has a built-in way to convert itself into this string format.

```kotlin
// Simplified from FilePath.kt
data class FilePath(...) {
    // ... other fields ...

    // Converts the FilePath object to its string representation
    override fun toString(): String {
        // Join the parts with '/', handling cases where some parts might be empty
        return "$objectType/$objectId/$directory/$name"
            .substringBefore("//") // Remove trailing slashes if directory/name are empty
            .removeSuffix("/")     // Remove trailing slash if name is empty
    }
}

// Example usage:
val filePath = FilePath("project", "proj-456", "docs", "report.pdf")
val pathString = filePath.toString() // Result: "project/proj-456/docs/report.pdf"
```

*   `override fun toString()`: This is a standard function in Kotlin. We're customizing it for `FilePath` to produce the slash-separated path.
*   The code carefully combines the parts and cleans up any extra slashes if parts like `directory` or `name` are missing.

**Code Snippet: Parsing a String Path**

The `FilePath` class can also do the reverse: parse a string path back into its structured parts.

```kotlin
// Simplified from FilePath.kt
data class FilePath(...) {
    companion object { // Helper functions related to the class
        // Creates a FilePath object from a slash-separated string
        fun from(path: String): FilePath {
            // Split the string by '/' into up to 4 parts
            val parts = path.split("/", limit = 4)
            // Make sure we always have 4 elements, padding with "" if needed
            val (objType, objId, dir, name) = parts.padEnd(4, "")
            return FilePath(objType, objId, dir, name)
        }
    }
    // ... other parts ...
}

// Example usage:
val pathString = "user/user-789/avatars/profile.png"
val filePath = FilePath.from(pathString)
// Result: filePath.objectType == "user"
//         filePath.objectId == "user-789"
//         filePath.directory == "avatars"
//         filePath.name == "profile.png"
```

*   `companion object`: A place to put functions related to the `FilePath` class itself, rather than a specific instance.
*   `from(path: String)`: Takes a string like `"a/b/c/d.txt"` and splits it to create the `FilePath` object.
*   `split("/", limit = 4)`: Breaks the string at the slashes, ensuring we get the four main parts correctly even if the filename itself contains slashes (though that's generally avoided).
*   `.padEnd(4, "")`: If the path is shorter (e.g., "project/proj-123/config.json"), it fills in the missing parts (like `directory`) with empty strings.

This ability to convert between the structured `FilePath` object and its simple string representation makes it easy to use the `FilePath` in your code while ensuring files are stored predictably in the underlying system.

## Conclusion

You've now learned about `FilePath`, the essential addressing system for files in `connect-fs`. You know it's composed of four key parts (`objectType`, `objectId`, `directory`, `name`) that provide a structured way to identify and organize files. We saw how to create a `FilePath` and how it's used in commands and events, and how it translates to a simple string path for storage.

With `FilePath`, you always know exactly where a file belongs, making file management clear and reliable.

But how does the system know *which* storage bucket to use, or how to generate the public URLs we saw in the `FileUploadedEvent`? That involves configuration. Let's explore that next!

Ready to configure? Let's go to [Chapter 3: Configuration (FsProperties)](03_configuration__fsproperties__.md)!

---

Generated by [AI Codebase Knowledge Builder](https://github.com/The-Pocket/Tutorial-Codebase-Knowledge)