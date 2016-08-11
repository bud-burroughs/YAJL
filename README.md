# YAJL

This is a Objective-C wrapper around the C-library [YAJL (Yet Another JSON
Library)](http://lloyd.github.com/yajl/), a SAX-style JSON parser.

You may want to parse your JSON file SAX-style if you are processing extremely
large files. Most JSON parsers read the entire file into memory before parsing it,
which only works with smaller files.

This repo is a fork of [yail-objc](https://github.com/gabriel/yajl-objc) and
adds Framework Targets for macOS and iOS. With this, you can easily use install
this framework with [Carthage](https://github.com/Carthage/Carthage.git).

## Install with Carthage

Install carthage with homebrew

    $ brew update
    $ brew install carthage

Integrate YAJL into your Xcode project by adding this to your `Cartfile`:

    github "sebcode/YAJL" "1.0"

Run carthage update to build the framework and drag the built framework into
your Xcode project.

## Quickstart Swift 2.1 Sample Code

```
import Cocoa
import YAJL

@NSApplicationMain
class AppDelegate: NSObject, NSApplicationDelegate {

    @IBOutlet weak var window: NSWindow!

    func applicationDidFinishLaunching(aNotification: NSNotification) {
        let parser = MyParser()
        parser.parseFile("/tmp/test.json")
    }

}

@objc
public class MyParser: NSObject, YAJLParserDelegate {

    public func parseFile(file: String) -> Bool {
        let parser = YAJLParser()
        parser.delegate = self

        guard let readHandle = NSFileHandle(forReadingAtPath: file) else {
            return false
        }

        var totalSize: UInt64 = 0
        var totalRead: Int64 = 0
        let bufSize = 1024 * 1024 * 5
        let start = NSDate()
        var didRead = 0

        do {
            let attr: NSDictionary = try NSFileManager.defaultManager().attributesOfItemAtPath(file)
            totalSize = attr.fileSize()
        } catch {
            return false
        }

        repeat {
            autoreleasepool {
                let buf = readHandle.readDataOfLength(bufSize)
                didRead = buf.length
                if buf.length > 0 {
                    parser.parse(buf)
                    totalRead += didRead
                    let percentComplete = Int((Float(totalRead) / Float(totalSize)) * 100.0)
                    NSLog("Progress: \(percentComplete)%")
                }
            }
        } while didRead > 0

        NSLog("Parse took: \(NSDate().timeIntervalSinceDate(start))s")

        return true
    }

    public func parserDidStartArray(parser: YAJLParser!) {
        NSLog("parserDidStartArray")
    }

    public func parserDidEndArray(parser: YAJLParser!) {
        NSLog("parserDidEndArray")
    }

    public func parserDidStartDictionary(parser: YAJLParser!) {
        NSLog("parserDidStartDictionary")
    }

    public func parserDidEndDictionary(parser: YAJLParser!) {
        NSLog("parserDidEndDictionary")
    }

    public func parser(parser: YAJLParser!, didMapKey key: String!) {
        NSLog("parser didMapKey \(key)")
    }

    public func parser(parser: YAJLParser!, didAdd value: AnyObject!) {
        NSLog("parser didAdd \(value)")
    }

}
```

## Credits

- Original repo: [yail-objc](https://github.com/gabriel/yajl-objc)
- This forked repo: [Sebastian Volland](https://github.com/sebcode)

## Links

- Original online [API documentation](http://gabriel.github.com/yajl-objc/)
- [YAJL C-library Website](http://lloyd.github.com/yajl/)

