## Onion Browser

[Official Site][official] | [Support][help]<br>
&copy; 2012 [Mike Tigas][miketigas] ([@mtigas](https://twitter.com/mtigas))<br>
[MIT License][license]

A minimal, open-source web browser for iOS that tunnels web traffic through
the [Tor network][tor]. See the [official site][official] for more details
and App Store links.

[official]: http://onionbrowser.com/
[help]: http://onionbrowser.com/help/
[miketigas]: http://mike.tig.as/
[license]: https://github.com/mtigas/iOS-OnionBrowser/blob/master/LICENSE

<a href="//d2p12wh0p3fo1n.cloudfront.net/files/20120413/004.png"><img src="//d2p12wh0p3fo1n.cloudfront.net/files/20120413/004-100.jpg" width="100"/></a>
<a href="//d2p12wh0p3fo1n.cloudfront.net/files/20120413/003.png"><img src="//d2p12wh0p3fo1n.cloudfront.net/files/20120413/003-100.jpg" width="100"/></a>
<a href="//d2p12wh0p3fo1n.cloudfront.net/files/20120413/002.png"><img src="//d2p12wh0p3fo1n.cloudfront.net/files/20120413/002-100.jpg" width="100"/></a>
<a href="//d2p12wh0p3fo1n.cloudfront.net/files/20120413/005.png"><img src="//d2p12wh0p3fo1n.cloudfront.net/files/20120413/005-100.jpg" width="100"/></a>
<br>
<a href="//d2p12wh0p3fo1n.cloudfront.net/files/20120413/p003.png"><img src="//d2p12wh0p3fo1n.cloudfront.net/files/20120413/p003-150.jpg" width="150"/></a>
<a href="//d2p12wh0p3fo1n.cloudfront.net/files/20120413/p002.png"><img src="//d2p12wh0p3fo1n.cloudfront.net/files/20120413/p002-150.jpg" width="150"/></a>
<a href="//d2p12wh0p3fo1n.cloudfront.net/files/20120413/p001.png"><img src="//d2p12wh0p3fo1n.cloudfront.net/files/20120413/p001-150.jpg" width="150"/></a>

#### Technical notes

* **OnionBrowser**: 1.2.6 (20120905.2)
* **Tor**: 0.2.3.20-rc (Aug 05 2012)
* **libevent**: 2.0.20-stable (Aug 23 2012)
* **OpenSSL**: 1.0.1c (May 10 2012)

The app, when compiled, contains static library versions of [Tor][tor] and it's
dependencies, [libevent][libevent] and [openssl][openssl].

[tor]: https://www.torproject.org/
[libevent]: http://libevent.org/
[openssl]: https://www.openssl.org/

The build scripts for Tor and these dependencies are based on
[build-libssl.sh][build_libssl] from [x2on/OpenSSL-for-iPhone][openssliphone].
The scripts are configured to compile universal binaries for armv7 and
i386 (for the iOS Simulator).

[build_libssl]: https://github.com/x2on/OpenSSL-for-iPhone/blob/c637f773a99810bb101169f8e534d0d6b09f3396/build-libssl.sh
[openssliphone]: https://github.com/x2on/OpenSSL-for-iPhone

The tor `build-tor.sh` script patches one file in Tor (`src/common/compat.c`)
to remove references to `ptrace()` and `_NSGetEnviron()`. This first is only used
for the `DisableDebuggerAttachment` feature (default: True) implemented in Tor
0.2.3.9-alpha. (See [changelog][tor_dev_changelog] and [manual][tor_dev_manual].)
`ptrace()` and `_NSGetEnviron()` calls are not allowed in App Store apps; apps
submitted with `ptrace()` symbols are rejected on upload by Apple's
auto-validation of the uploaded binary. (The `_NSGetEnviron()` code does not
even compile when using iPhoneSDK due to that function being undefined.)
See the patch files in `build-patches/` if you are interested in the changes.

[tor_dev_changelog]: https://gitweb.torproject.org/tor.git/blob/tor-0.2.3.17-beta:/ChangeLog
[tor_dev_manual]: https://www.torproject.org/docs/tor-manual-dev.html.en

0.2.3.17-beta introduced compiler and linker "hardening" ([Tor ticket 5210][ticket5210]),
which is incompatible with the iOS Device build chain.  The app (when building
for iOS devices) is configured with `--disable-gcc-hardening --disable-linker-hardening`
to get around this issue. (Due to the isolation of executable code on iOS devices,
this should not cause a significant change in security.)

[ticket5210]: https://trac.torproject.org/projects/tor/ticket/5210

Because iOS applications cannot launch subprocesses or otherwise execute other
binaries, the tor client is run in-process in a `NSThread` subclass which
executes the `tor_main()` function (as an external `tor` executable would)
and attempts to safely wrap Tor within the app. (`libor.a` and
`libtor.a`, intermediate binaries created when compiling Tor, are used to
provide Tor.) Side-effects of this method have not yet been fully evaluated.
Management of most tor functionality (status checks, reloading tor on connection
changes) is handled by accessing the Tor control port in an internal, telnet-like
session from the `AppDelegate`.

The app uses a `NSURLProtocol` subclass (`ProxyURLProtocol`), registered to
handle HTTP/HTTPS requests. That protocol uses the `CKHTTPConnection` class
which nearly matches the `NSURLConnection` class, providing wrappers and access
to the underlying `CFHTTP` Core Framework connection bits. This connection
class is where SOCKS5 connectivity is enabled. (Because we are using SOCKS5,
DNS requests are sent over the Tor network, as well.)

(I had WireShark packet logs to support the claim that this app protects all
HTTP/HTTPS/DNS traffic in the browser, but seem to have misplaced them. You'll
have to take my word for it or run your own tests.)

The app uses [Automatic Reference Counting (ARC)][arc] and was developed against
iOS 5.X. (It *may* work when building against iOS 4.X, since most of the ARC
behavior exists in that older SDK, with the notable exception of weakrefs.)

[arc]: https://developer.apple.com/library/ios/releasenotes/ObjectiveC/RN-TransitioningToARC/index.html

## Building

1. Check Xcode version
2. Build dependencies via command-line
3. Build application in XCode

### Check Xcode version

Double-check that the "currently selected" Xcode Tools correspond to the version
of Xcode you have installed:

    xcode-select -print-path

For the newer Xcode 4.3+ installed via the App Store, the directory should be
`/Applications/Xcode.app/Contents/Developer`, and not the straight `/Developer`
(used by Xcode 4.2 and earlier). If you have both copies of Xcode installed
(or if you have updated to Xcode 4.3 but `/Developer` still shows), do this:

    sudo xcode-select -switch /Applications/Xcode.app/Contents/Developer

### Building dependencies

`cd` to the root directory of this repository and then run these commands in
the following order to build the dependencies. (This can take anywhere between
five and thirty minutes depending on your system speed.)

    bash build-libssl.sh
    bash build-libevent.sh
    bash build-tor.sh
    bash OnionBrowser/icon/install.sh

This should create a `dependencies` directory in the root of the repository,
containing the statically-compiled library files.

### Build OnionBrowser.xcodeproj in Xcode

Open `OnionBrowser/OnionBrowser.xcodeproj`. You should be
able to compile and run the application at this point. (The app is compatible
with armv7 and i386 targets, meaning that all iOS 5.0 devices and the
iPhone/iPad Simulators should be able to run the application.)
