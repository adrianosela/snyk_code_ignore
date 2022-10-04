# snyk_code_ignore

> Note: Tested with Snyk CLI version 1.1019.0

A thin wrapper around the Snyk CLI that allows ignoring code issues with in-line comments

## Why? The Motivation Behind This

**It is currently not possible to ignore first-party code issues when using the Snyk CLI's `snyk code test` functionality**. This is a problem, because like any other SAST/SCA tool, Snyk Code reports often false positives. 

What folks end up doing today to ignore a false positive finding from appearing in results from the Snyk CLI and get on with their lives is they permanently exclude the file from code scans altogether using an `exclude` clause in the `.snyk` file (permanently because the `exclude` clause does not support an `expiry`).

The problem is even worse if you are using the Snyk CLI (or the Snyk Jenkins plugin, which uses the Snyk CLI under-the-hood) to "break-the-build" whenever there are issues introduced to the code base. In this scenario, you either exclude the file from analysis as described above, or you see the failure regularly and end up overriding failed CI builds.

### Example

Consider the following issue for example:

```
 ✗ [Medium] Use of Password Hash With Insufficient Computational Effort
   Path: cache.go, line 294
   Info: The SHA1 hash (used in crypto.sha1.New) is insecure. Consider changing it to a secure hash algorithm
```

The finding above is complaining about the use of a weak hashing algorithm -- however, upon inspecting the code we find that these hashes are being used as keys for a key-value storage (not for hasing passwords). So we have ourselves a false positive finding.

So now... what do we do to get the Snyk CLI to stop reporting this as an issue? Tough luck... there's nothing you can do short of excluding the file from analysis. This means we would not be able to catch issues introduced to the file in the future.

**THIS SUCKS -- there's gotta be a better way right?**

## Usage

### Installation

Download the script to your binaries path and make it executable:
```
curl https://raw.githubusercontent.com/adrianosela/snyk_code_ignore/main/snyk_code_ignore --output /usr/local/bin/snyk_code_ignore
chmod +x /usr/local/bin/snyk_code_ignore
```

### Ignoring an Issue

To ignore an issue simply annotate the offending line or the line above it with:

```
snyk:ignore:${FINDING_TITLE}
```

Recall the example finding shown before:
```
 ✗ [Medium] Use of Password Hash With Insufficient Computational Effort
   Path: cache.go, line 294
   Info: The SHA1 hash (used in crypto.sha1.New) is insecure. Consider changing it to a secure hash algorithm
```

We can ignore it going forward by adding a `snyk:ignore` comment in the offending line, or the line directly above it as follows:

```
        // snyk:ignore:Use of Password Hash With Insufficient Computational Effort
        hash := sha1.New()
        hash([]byte(marshalledObject))
        cacheKey := hex.EncodeToString(hash.Sum(nil))
```

Further runs of the program will exclude this issue from the list of findings.

### Running the Program

Run the script as you normally would run `snyk code test`. The program is a simple wrapper around this command. All arguments and flags are passed down to the `snyk code test` command.

> Note: At this time, only the `--severity-threshold` flag is supported. Any flags that modify the shape of the output will cause the program to not work as intended (the program relies on the output of the `snyk code test` command looking a certain specific way).

#### Example

```
10:40 $ snyk_code_ignore mockpath/ --severity-threshold=medium

Testing mockpath/ ...

 ✗ [Medium] Use of Password Hash With Insufficient Computational Effort
   Path: mockservice/handler.go, line 143
   Info: The SHA1 hash (used in crypto.sha1.New) is insecure. Consider changing it to a secure hash algorithm


✔ Test completed

Organization:      mock-organizaion
Test type:         Static code analysis
Project path:      mockpath/

```
