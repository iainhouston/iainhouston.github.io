---
title: A Swift Jekyll Assistant
date:  2025-05-05 07-02-52
layout: post
comments: true
categories: ['Scripting']
---

Well, this has been a fascinating project: I was looking back at my [AppleScript Jekyll Assistant]({% post_url 2020-02-03-Applescript_Jekyll_assistant_pt_5 %}) and thinking that "this is so much more complicated than it neeeds to be". So I rewrote it in Swift using the [Swift Argument Parser](https://github.com/apple/swift-argument-parser).  

Quite apart from anything else, I really love the conditional `let` (`if let`): allows for a really quite elegant, readable solution.

I had written command line scripts in Swiftusing the Swift Argument Parser but was a bit short of time to study the documentation again and to remember the Swift development ecosystem. So ChatGPT came to the rescue and helped me write the following.   

So my conclusion from this experience is that AI has definitely been a “productivity enhancer” here. I know my way around many different programming languages but obviously am not an expert in those APIs that I rarely use: take the `expandingTildeInPath` property getter in the Foundation APIs for example. And getting the regular expressions just right is something that usually would have taken me a lot of time. So ChatGPT helped me with the important and unfamiliar detail and I think you’ll agree we came up with a pretty elegant solution.

```swift
#!/usr/bin/env swift

import Foundation
import ArgumentParser

struct CLI: ParsableCommand {
    static let configuration = CommandConfiguration(
        abstract: "A command-line tool that processes a category and description and generates a markdown file."
    )

    @Option(name: [.short, .long], help: "The category of the item.")
    var category: String

    @Option(name: [.short, .long], help: "The description of the item.")
    var description: String

    @Option(name: [.short, .long], help: "Optional: directory to write output markdown file. Overrides and sets persistent path.")
    var outputPath: String?

    func run() throws {
        let fileManager = FileManager.default
        let homeDir = fileManager.homeDirectoryForCurrentUser
        let configURL = homeDir.appendingPathComponent(".mycli_config")

        // Determine target directory for output
        let targetDirectory: String
        if let providedPath = outputPath {
            // Expand ~ and create directory
            let expanded = (providedPath as NSString).expandingTildeInPath
            try fileManager.createDirectory(atPath: expanded, withIntermediateDirectories: true, attributes: nil)
            // Save persistent path
            try expanded.write(to: configURL, atomically: true, encoding: .utf8)
            targetDirectory = expanded
        } else if fileManager.fileExists(atPath: configURL.path) {
            // Load saved path
            let savedPath = try String(contentsOf: configURL, encoding: .utf8)
                .trimmingCharacters(in: .whitespacesAndNewlines)
            targetDirectory = savedPath
        } else {
            throw ValidationError("No output path provided and no persistent path found. Use --output-path to set one.")
        }

        // Print received inputs
        print("Category: \(category)")
        print("Description: \(description)")
        print("Output directory: \(targetDirectory)")

        // Prepare date strings
        let fileDateFormatter = DateFormatter()
        fileDateFormatter.dateFormat = "yyyy-MM-dd"
        let dateString = fileDateFormatter.string(from: Date())

        let timestampFormatter = DateFormatter()
        timestampFormatter.dateFormat = "yyyy-MM-dd HH-mm-ss"
        let timestampString = timestampFormatter.string(from: Date())

        // Sanitize description for filename
        let sanitizedDescription = description
            .trimmingCharacters(in: .whitespacesAndNewlines)
            .replacingOccurrences(of: "\\s+", with: "_", options: .regularExpression)
            .replacingOccurrences(of: "[^A-Za-z0-9_-]", with: "", options: .regularExpression)

        // Build filename and paths
        let baseFilename = "\(dateString)-\(sanitizedDescription)"
        let outputFilename = "\(baseFilename).markdown"
        let fileURL = URL(fileURLWithPath: targetDirectory)
            .appendingPathComponent(outputFilename)

        // Prepare title for front matter (retain spaces)
        let titleString = description.replacingOccurrences(of: "_", with: " ")

        // Build markdown content
        let markdownContent = """
        ---
        title: \(titleString)
        date:  \(timestampString)
        layout: post
        comments: true
        categories: ['\(category)']
        ---
        """

        // Write the markdown file
        try markdownContent.write(to: fileURL, atomically: true, encoding: .utf8)
        print("Created markdown file: \(outputFilename) at \(targetDirectory)")

        // Open the file in Visual Studio Code
        do {
            let openProcess = Process()
            openProcess.executableURL = URL(fileURLWithPath: "/usr/bin/open")
            openProcess.arguments = ["-a", "/Applications/Visual Studio Code.app", fileURL.path]
            try openProcess.run()
        } catch {
            print("Failed to open file in VS Code: \(error)")
        }
    }
}

// Entry point
CLI.main()
```