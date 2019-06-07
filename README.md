ModernHttpClient
================

Available on NuGet: https://www.nuget.org/packages/modernhttpclient-updated/ [![NuGet](https://img.shields.io/nuget/v/modernhttpclient-updated.svg?label=NuGet)](https://www.nuget.org/packages/modernhttpclient-updated/)

This library brings the latest platform-specific networking libraries to Xamarin applications via a custom HttpClient handler.

This is made possible by:

* On iOS, [NSURLSession](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSURLSession_class/Introduction/Introduction.html)
* On Android, via [OkHttp3](http://square.github.io/okhttp/)
* On UWP, via a custom HttpClientHandler.

## How can I use this from shared code?

Just reference ModernHttpClient in your .Net Standard or Portable Library, and it will use the correct version on all platforms.

### What's new?

Use SSLConfig parameter to provide server certificate chain public keys and a client certificate for Mutual TLS Authentication.

TLS 1.2 has been enforced.

Read why here:

https://www.brillianceweb.com/resources/answers-to-7-common-questions-about-upgrading-to-tls-1.2/

### Usage

Here's how it works:

```cs
readonly static HttpClient client = new HttpClient(new NativeMessageHandler(false, new SSLConfig()
{
    Pins = new List<Pin>()
    {
        new Pin()
        {
            Hostname = "gorest.co.in",
            PublicKeys = new []
            {
                "sha256/MCBrX+0kgfNc/qacknAJ5nojbFIx7kBSJSmXKjJviIg=",
                "sha256/YLh1dUR9y6Kja30RrAn7JKnbQG/uEtLMkBgFF2Fuihg=",
                "sha256/Vjs8r4z+80wjNcr1YKepWQboSIRi63WsWXhIMN+eWys="
            }
        }
    },
    ClientCertificate = new ClientCertificate()
    {
        RawData = "PFX_DATA",
        Passphrase = "PFX_PASSPHRASE"
    }
})
{
    DisableCaching = true,
    Timeout = new TimeSpan(0, 0, 9)
});
```

If server certificate chain public keys are not provided, the basic server certificate verification will be done.

### How to obtain server certificate chain public keys?

1. In your Android project add Square.OkHttp3 Nuget Package.
2. In your Main Activity, add a button and on its click event run this code:

```cs
var hostname = "gorest.co.in";
var certificatePinner = new Square.OkHttp3.CertificatePinner.Builder()
    .Add(hostname, "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=")
    .Build();
var client = new OkHttpClient.Builder()
    .CertificatePinner(certificatePinner)
    .Build();
 
var request = new Request.Builder()
    .Url("https://" + hostname)
    .Build();
 
var call = client.NewCall(request);
var response = await call.ExecuteAsync();
```

As expected, this fails with a certificate pinning exception:

```cs
Certificate pinning failure!
  Peer certificate chain:
    sha256/MCBrX+0kgfNc/qacknAJ5nojbFIx7kBSJSmXKjJviIg=: CN=gorest.co.in
    sha256/YLh1dUR9y6Kja30RrAn7JKnbQG/uEtLMkBgFF2Fuihg=: CN=Let's Encrypt Authority X3,O=Let's Encrypt,C=US
    sha256/Vjs8r4z+80wjNcr1YKepWQboSIRi63WsWXhIMN+eWys=: CN=DST Root CA X3,O=Digital Signature Trust Co.
  Pinned certificates for gorest.co.in:
    sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=
```

3. Follow up by pasting the public key hashes from the exception into the NativeMessageHandler certificate pinner's configuration.

### Client certificate as Base64

A certificate in pfx format can be created from client.crt, server.key and a passphrase using openssl:

```
openssl pkcs12 -export -out client.pfx -inkey server.key -in client.crt
```

A client.crt can be created following this tutorial: https://gist.github.com/mtigas/952344

The pfx certificate can be converted to Base64 using PowerShell:

```
$fileContentBytes = get-content 'path-to\client.pfx' -Encoding Byte
[System.Convert]::ToBase64String($fileContentBytes) | Out-File 'path-to\pfx-bytes.txt'
```

Or, from a terminal window on a Mac:

```
base64 -i path-to/client.pfx -o path-to/pfx-bytes.txt
```

### How to use NativeCookieHandler?

SetCookie before making the http call and they will be added to Cookie header in the native client:

```cs
NativeCookieHandler cookieHandler = new NativeCookieHandler();

readonly static HttpClient client = new HttpClient(new NativeMessageHandler(false, new SSLConfig()
{
    Pins = new List<Pin>()
    {
        new Pin()
        {
            Hostname = "gorest.co.in",
            PublicKeys = new []
            {
                "sha256/MCBrX+0kgfNc/qacknAJ5nojbFIx7kBSJSmXKjJviIg=",
                "sha256/YLh1dUR9y6Kja30RrAn7JKnbQG/uEtLMkBgFF2Fuihg=",
                "sha256/Vjs8r4z+80wjNcr1YKepWQboSIRi63WsWXhIMN+eWys="
            }
        }
    },
    ClientCertificate = new ClientCertificate()
    {
        RawData = "PFX_DATA",
        Passphrase = "PFX_PASSPHRASE"
    }
},
cookieHandler)
{
    DisableCaching = true,
    Timeout = new TimeSpan(0, 0, 9)
});

var cookie = new Cookie("cookie1", "value1", "/", "reqres.in");
cookieHandler.SetCookie(cookie);

var response = await client.GetAsync(new Uri("https://reqres.in"));
```

## NativeCookieHandler methods

```SetCookies(IEnumerable<Cookie> cookies)```: set native cookies

```DeleteCookies()```: delete all native cookies.

```SetCookie(Cookie cookie)```: set a native cookie.

```DeleteCookie(Cookie cookie)```: delete a native cookie.

## How to use a Web Proxy?

Just provide the Web Proxy configuration as part of the ctor:

```cs
readonly static HttpClient client = new HttpClient(new NativeMessageHandler(false, new SSLConfig()
{
    Pins = new List<Pin>()
    {
        new Pin()
        {
            Hostname = "gorest.co.in",
            PublicKeys = new []
            {
                "sha256/MCBrX+0kgfNc/qacknAJ5nojbFIx7kBSJSmXKjJviIg=",
                "sha256/YLh1dUR9y6Kja30RrAn7JKnbQG/uEtLMkBgFF2Fuihg=",
                "sha256/Vjs8r4z+80wjNcr1YKepWQboSIRi63WsWXhIMN+eWys="
            }
        }
    },
    ClientCertificate = new ClientCertificate()
    {
        RawData = "PFX_DATA",
        Passphrase = "PFX_PASSPHRASE"
    }
},
cookieHandler,
new System.Net.WebProxy("127.0.0.1:80", false))
{
    DisableCaching = true,
    Timeout = new TimeSpan(0, 0, 9)
});
```

If the Web Proxy address is not reachable, you will get the following exceptions:

iOS:

```
KCFErrorDomainCFNetwork error 310: cannot find the proxy server.
```

Android:

```
System.Net.Http.HttpRequestException
Failed to connect to /127.0.0.1:80
```

UWP:

```
Exception: The text associated with this error code could not be found.
The server name or address could not be resolved
```

#### Release Notes

3.2.0

Code refactoring.

Adding support for Mutual TLS Authentication

Enforcing TLS1.2

Adding support for web proxy.

[iOS] Removing minimumSSLProtocol static property.

[Android] Removing verifyHostnameCallback static property.

[Android] Removing customTrustManager static property.

2.7.2

[Android] Handshake failed (adding customTrustManager static property) #11

2.7.1

[Android] MissingMethodException Method 'ModernHttpClient.NativeMessageHandler..ctor' not found. #9

[iOS] Removing minimumSSLProtocol from NativeMessageHandler ctor

[UWP] Exception on UWP with Xamarin Forms #3

2.7.0
      
[Update] Migrating to a multi-target project
      
[Android] Calling HttpClient methods should throw .Net Exception when fail #5
      
[Android] VerifyHostnameCallback parameter function on constructor (NativeMessageHandler - Android) when customSSLVerification is true #6
      
[Android] ReasonPhrase is empty under HTTPS #8

2.6.0

[Update] Adding support for UWP

[Update] Adding support for netstandard 2.0

2.5.3

[Update] Cookies set with the native handler will be merged into the Cookie header

2.5.1

[Android] NativeCookieHandler, if provided, is set as the default CookieJar for OkHttpClient

[Update] Adding DeleteCookies, SetCookie and DeleteCookie to NativeCookieHandler

2.5.0

[Android] Updating to Square.OkHttp3

2.4.9

[Android] Calling HttpClient methods should throw .Net Exception when fail #5

[Android] MissingMethodException Method 'ModernHttpClient.NativeMessageHandler..ctor' not found. #9

[Android] VerifyHostnameCallback parameter function on constructor when customSSLVerification is true #6

[Android] ReasonPhrase is empty under HTTPS #8

[Android] Handshake failed (adding customTrustManager static property) #11

[iOS] Removing minimumSSLProtocol from NativeMessageHandler ctor

2.4.7

[Update] Cookies set with the native handler will be merged into the Cookie header

2.4.5

[Android] NativeCookieHandler, if provided, is set as the default cookie handler for OkHttpClient

[Update] Adding DeleteCookies, SetCookie and DeleteCookie to NativeCookieHandler

2.4.4

[Android] SIGABRT after UnknownHostException #229

[iOS] Updating obsolete NSUrlSessionDelegate to INSUrlSessionDelegate

[Update] Adding EnableUntrustedCertificates to support self-signed certificates

2.4.3

[Update] Adding Timeout property

[Android] Updating to Square.OkHttp 2.7.5

[Android] Timeout value is not respected on Android #192
