# Notes on reverse engineering the qiui app

Everything here is [apache 2 licensed](https://www.apache.org/licenses/LICENSE-2.0)

The qiui app is very heavily obfuscated. The android apk is a shell.
It largely uses a function `a.aau.c` to replace constant strings with
encrypted strings. This function doesn't appear to be any standard
algorithm I could find, but there's an easy way to run it. Write a java file like:

```
import a.auu.a;

public class Main {
  public static void main(String[] args) {
    String[] entries = new String[] {
      "AAo3CQAAFgoAEiMOBgsqIAYXDgE=",
"AwAADQ4XRQ==",
"AxA5EA==",
"CAwRCQVT",
"CQwCAA9TBiELAAAZB0UnFlQLFB8J",
"CQwCAA9TCScHBgQTCkUnFlQACAcNKxdUCxQfCW4KBkUEHhU6HA==",
    };
    for (String s : entries) {
      String res = a.c(s);
      System.out.println("original: " + s + " decoded: " + res);
    }
  }
}
```

and create a fake a/aau/a.java like:

```
package a.auu;

public class a {
  public static String c(String str)  {
    return str;
  }
}
```

then:

```
#!/bin/bash
javac Main.java
~/Library/Android/sdk/build-tools/30.0.3/dx --dex --output foo.dex Main.class
```
(you need to replace `...` with the path to the app on your phone)

```
#!/bin/bash
set -euxo pipefail
adb push foo.dex /sdcard/foo.dex
adb shell su -c "dalvikvm -cp /data/app/.../cellmate.qiui.com.../base.apk:/sdcard/foo.dex Main"
```
and you can dump those strings.

The function `attachBaseContext` in `MyApplication` is doing most of the work in loading the code
we're working on pulling that out
