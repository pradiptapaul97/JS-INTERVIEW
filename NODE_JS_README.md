# How do you handle large file data efficiently in Node.js?

## Answer

When dealing with large files in Node.js, I avoid using `fs.readFile()` because it loads the entire file into memory before processing it. For very large files, this can cause high memory consumption or even crash the application.

Instead, I use **Streams**, which process data in smaller chunks. Streams are memory-efficient because only a portion of the file is kept in memory at any given time.

---

## Bad Approach

```javascript
const fs = require("fs");

fs.readFile("large-file.csv", "utf8", (err, data) => {
  console.log(data);
});
```

### Problem
```
1 GB file
    ↓
Load entire 1 GB into RAM
    ↓
Process
```

If multiple users upload large files simultaneously, memory usage can become very high.

---

## Good Approach: Streams

```javascript
const fs = require("fs");

const readStream = fs.createReadStream("large-file.csv", {
  encoding: "utf8",
});

readStream.on("data", (chunk) => {
  console.log(`Received ${chunk.length} bytes`);
});

readStream.on("end", () => {
  console.log("File processing completed");
});
```

### Flow
```
1 GB file
    ↓
64 KB chunk
    ↓
Process
    ↓
Next 64 KB chunk
```

Memory remains almost constant regardless of file size.

---

## Real-World Example: File Upload

### Instead of:

```javascript
app.post("/upload", (req, res) => {
  // Read entire file into memory ❌
});
```

### Use:

```javascript
const fs = require("fs");

app.post("/upload", (req, res) => {
  const writeStream = fs.createWriteStream("./uploads/file.mp4");

  req.pipe(writeStream);

  writeStream.on("finish", () => {
    res.send("Upload completed");
  });
});
```

This streams the uploaded file directly to disk.

---

## Copying Large Files Efficiently

```javascript
const fs = require("fs");

const readStream = fs.createReadStream("source.zip");
const writeStream = fs.createWriteStream("destination.zip");

readStream.pipe(writeStream);
```

### Benefits:

- **Low memory usage**
- **Faster processing**
- **Built-in backpressure handling**
