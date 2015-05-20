# iStringsCheck

iStringsCheck is an utility to verify completeness of .strings (iOS & OS X) translations files and the count of their format attributes.

## Motivation

It is common to crowdsource translations using platforms like Transifex. Unfortunately, Transifex [does not verify that translations do contain the same number of occurences of a given formatter](https://twitter.com/transifex/status/601092349381357568). This is particularly an issue with Objective-C and C-related languages that do not provide any kind of memory safety guarantees on formatted strings.

## Usage
```
istringscheck <source_strings> <translations_dir>
```
## Verifications

Currently iStringsCheck only verifies whether all localization files contain all of the localization keys and that for a given key, the number of occurences of the formatter %@ is the same as a source language.

## About format strings attacks
If you are taking input from a user or other untrusted source and displaying it, you need to be careful that your display routines do not process format strings received from the untrusted source. For example, in the following code the syslog standard C library function is used to write a received HTTP request to the system log. Because the syslog function processes format strings, it will process any format strings included in the input packet:

```
/* receiving http packet */
int size = recv(fd, pktBuf, sizeof(pktBuf), 0);
if (size) {
syslog(LOG_INFO, "Received new HTTP request!");
syslog(LOG_INFO, pktBuf);
}
```
Many format strings can cause problems for applications. For example, suppose an attacker passes the following string in the input packet:

```
"AAAA%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%n"
```

This string retrieves eight items from the stack. Assuming that the format string itself is stored on the stack, depending on the structure of the stack, this might effectively move the stack pointer back to the beginning of the format string. Then the %n token would cause the print function to take the number of bytes written so far and write that value to the memory address stored in the next parameter, which happens to be the format string. Thus, assuming a 32-bit architecture, the AAAA in the format string itself would be treated as the pointer value 0x41414141, and the value at that address would be overwritten with the number 76.

Doing this will usually cause a crash the next time the system has to access that memory location, but by using a string carefully crafted for a specific device and operating system, the attacker can write arbitrary data to any location. See the manual page for printf for a full description of format string syntax.

Source: [Secure Coding Guide](https://developer.apple.com/library/mac/documentation/Security/Conceptual/SecureCodingGuide/Articles/ValidatingInput.html)
