![NAXSI Logo](.assets/naxsi_logo.png)

# Nginx with NAXSI Module

This container set provides a custom built Nginx with NAXSI Web Application Firewall module.

Originally I built this from the ground up using `alpine:311` I got it running, but then ran into a problem when I wanted to shrink the image using a multi-stage build approach. Using a single build stage I ended up with a docker image size of 406MB. After rethinking and rebuilding using `nginx:alpine` I was able to get a multi-stage build and the image size shrank to a more palatable 128MB.

The problem with using `nginx:alpine` is that it does not support python2. The NAXSI version available for download requires python2 to build.

To resolve the python version issue i had to download the 1.3 version of NAXSI from their repo and edit a few of the files to make them syntactically correct for python3. This wasn't too difficult. There were plenty of `print` statements to be changed to `print()` statements and one literal comparison of `is not` to change to `!=`. Whilst syntactically correct it now build and works, but not being a python expert, or familiar with the NAXSI project coding, this may cause issues later.

## Testing

The rules in the default.conf location:

```nginx
    SecRulesEnabled; # enable naxsi
    # LearningMode; # enable learning mode
    LibInjectionSql; # enable libinjection support for SQLI
    LibInjectionXss; # enable libinjection support for XSS

    DeniedUrl "/denied.html"; # the location where naxsi will redirect the request when it is blocked
    CheckRule "$SQL >= 8" BLOCK; # the action to take when the $SQL score is superior or equal to 8
    CheckRule "$RFI >= 8" BLOCK;
    CheckRule "$TRAVERSAL >= 5" BLOCK;
    CheckRule "$UPLOAD >= 5" BLOCK;
    CheckRule "$XSS >= 8" BLOCK;
```

When triggered will cause a redirect to `/denied.html`.

To trigger it to prove this you can use an SQL injection attack in your URL, eg.

[http://localhost:8080/index.html?table=--delete%20from%20table%20where%201=1;](http://localhost:8080/index.html?table=--delete%20from%20table%20where%201=1;)

## Usage

Replace the `default.conf` file by mounting you own on top of it.
