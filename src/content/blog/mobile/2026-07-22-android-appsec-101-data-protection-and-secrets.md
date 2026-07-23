---
author: Samuel Marques
pubDatetime: 2026-07-22
modDatetime: 2026-07-22
title: Android AppSec 101 - Data Protection & Secrets
ogImage: Android AppSec 101 - Data Protection & Secrets
slug: android-appsec-101-part-1
featured: false
draft: false
tags:
  - mobile
  - android
description: This post is Part 1 of the Android AppSec 101 Series, where we
  analyze real-world mobile vulnerabilities, inspect source code, and implement
  secure fixes using the Allsafe laboratory target.
---
# Android AppSec 101 - Data Protection & Secrets

This post is **Part 1** of the **Android AppSec 101 Series**, where we analyze real-world mobile vulnerabilities, inspect source code, and implement secure fixes using the [Allsafe](https://github.com/t0thkr1s/allsafe-android) laboratory target.

- **[Part 0: Introduction](https://samuelmarques.dev/posts/android-appsec-101-part-0/)**
- **Part 1: Data Protection & Secrets** *(You are here)*
- **[Part 2: Android Component & IPC Security](https://samuelmarques.dev/posts/android-appsec-101-part-2/)**
- **[Part 3: Injections & Code Execution](https://samuelmarques.dev/posts/android-appsec-101-part-3/)**
- **[Part 4: Client-Side Bypasses](https://samuelmarques.dev/posts/android-appsec-101-part-4/)**
- **[Part 5: RE & Binary Patching](https://samuelmarques.dev/posts/android-appsec-101-part-5/)**

## Executive Summary

Mobile applications frequently handle sensitive data — from API keys and authentication tokens to personal user information. When developers rely on client-side storage without proper encryption or leave debugging tools active in production, attackers can extract this data with minimal effort.

In Part 1 of this series, we examine five fundamental data protection vulnerabilities inside [Allsafe](https://github.com/t0thkr1s/allsafe-android), analyze the decompiled source code using JADX-GUI, and demonstrate how to remediate each flaw using Android security best practices.

## Module 1: Insecure Logging

> **Quick Attack Preview**
>
> Enter `password123` in the challenge and press **Enter**. While monitoring `adb logcat --pid <PID> | grep ALLSAFE`, the secret immediately appears in plaintext. Let's reproduce the issue first, then investigate why it happens and how to fix it.

During development, engineers use logging mechanisms to trace execution flow and debug errors. If these calls remain active in production release builds, any process or connected system with access to system logs (`logcat`) can read cleartext sensitive data.

Prior to Android 4.1 (API level 16), any privileged application with `READ_LOGS` permission could inspect logs from every process. Modern Android versions restrict this access considerably, but logs remain visible through ADB during debugging, rooted devices, engineering builds, OEM debugging tools, crash-reporting frameworks, and developer oversight. Consequently, logging secrets is still considered an information disclosure vulnerability.

In the OWASP Mobile Security framework, this weakness is formally categorized under **[MASWE-0001: Insertion of Sensitive Data into Logs](https://mas.owasp.org/MASWE-0001/)** (mapping to global **CWE-532**). It directly violates the **MASVS-STORAGE** security standard.

[screenshot of insecure logging page]

### Dynamic Analysis

We can connect to the device via ADB and use `logcat` to monitor the logs of the `allsafe` application.

We start by inspecting the running processes on the device to identify the PID of the `allsafe` application.

```bash
# Get all running processes
adb shell ps

# Search for "ALLSAFE" in the process list
adb shell ps | grep "ALLSAFE"
```

Output:

```
USER           PID  PPID     VSZ    RSS WCHAN            ADDR S NAME
u0_a184       4700   319 13821972 135664 0                  0 S infosecadventures.allsafe
```

Now, we can use `logcat` to monitor the logs of the `allsafe` application. And if we input something in the application, we should see some logs in the output.

```bash
adb logcat --pid 4700 | grep "ALLSAFE"

# Or in case you don't have grep available
adb shell "logcat --pid 4700 | grep ALLSAFE"
```

[screenshot of logcat output]

### Static Analysis

We can start by using [JADX-GUI](https://github.com/skylot/jadx) to decompile the APK file and inspect the source code. Then, we look for any potential sources of sensitive data leaks. In this case, we are looking for `Log.d`, `Log.e`, `Log.i`, `Log.v`, and `Log.w` calls that might be logging sensitive information. In this specific case we can see an instance of `Log.d` in the `infosecadventures.allsafe.challenges.InsecureLogging.java` file.

```java
package infosecadventures.allsafe.challenges;

import android.os.Bundle;
import android.text.Editable;
import android.util.Log;
import android.view.KeyEvent;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;
import androidx.fragment.app.Fragment;
import com.google.android.material.textfield.TextInputEditText;
import infosecadventures.allsafe.R;
import java.util.Objects;

public class InsecureLogging extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_insecure_logging, container, false);
        setHasOptionsMenu(true);
        final TextInputEditText secret = (TextInputEditText) view.findViewById(R.id.secret);
        secret.setOnEditorActionListener(new TextView.OnEditorActionListener() {
            @Override
            public final boolean onEditorAction(TextView textView, int i, KeyEvent keyEvent) {
                return InsecureLogging.lambda$onCreateView$0(secret, textView, i, keyEvent);
            }
        });
        return view;
    }

    static boolean lambda$onCreateView$0(TextInputEditText secret, TextView v, int actionId, KeyEvent event) {
        if (actionId == 6 && !((Editable) Objects.requireNonNull(secret.getText())).toString().equals("")) {
            Log.d("ALLSAFE", "User entered secret: " + secret.getText().toString());
            return false;
        }
        return false;
    }
}
```

#### Code Breakdown

```java
public class InsecureLogging extends Fragment {

    // onCreateView lifecycle method (called when the fragment is created)
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {

        // Inflate the layout for this fragment
        View view = inflater.inflate(R.layout.fragment_insecure_logging, container, false);

        // Register this fragment to receive options menu callbacks
        setHasOptionsMenu(true);

        // Locate the text entry field for the user input (which is a "secret")
        final TextInputEditText secret = (TextInputEditText) view.findViewById(R.id.secret);

        // Set an OnEditorActionListener to handle text input
        secret.setOnEditorActionListener(new TextView.OnEditorActionListener() {
            @Override
            public final boolean onEditorAction(TextView textView, int i, KeyEvent keyEvent) {
                return InsecureLogging.lambda$onCreateView$0(secret, textView, i, keyEvent);
            }
        });

        // Return the view
        return view;
    }

    // Lambda method to handle text input
    static boolean lambda$onCreateView$0(TextInputEditText secret, TextView v, int actionId, KeyEvent event) {
        // Check if the actionId is 6 (IME_ACTION_DONE - user pressed enter key) and the text is not empty
        if (actionId == 6 && !((Editable) Objects.requireNonNull(secret.getText())).toString().equals("")) {
            // Log the secret to the system logs
            Log.d("ALLSAFE", "User entered secret: " + secret.getText().toString());
            return false;
        }
        // Return false to indicate that the event was not handled
        return false;
    }
}
```

In summary, if the user inputed something that is not empty into the secret text input field, and then pressed enter, the application will log the input to the system logs.

### Remediation & Code Fix

There are some ways we can fix this code. The point is that if you are developing an mobile application, you should not be logging sensitive information to the system logs. It's a security risk and a bad practice.

To remediate insecure logging vulnerabilities, developers should simply avoid logging sensitive information to the system logs in production environments. We can do that by preventing logging at all costs (removing all log calls in release mode), use a logging framework ([Timber](https://github.com/jakewharton/timber) in Java, [Logger](https://pub.dev/packages/logger) in Flutter) that supports conditional logging, or setting R8 proguard rules to remove all log levels except warning and error.

#### 1. Primary Remediation: Remove or Redact Sensitive Logs

The most direct fix is to remove any log call that outputs sensitive user inputs, credentials, or PII. If operational logging is necessary during development, ensure the output is redacted or masked:

```java
// BEFORE (Vulnerable):
Log.d("ALLSAFE", "User entered secret: " + secret.getText().toString());

// AFTER (Remediated):
// Completely remove the log, or restrict to debug mode with non-sensitive metadata:
if (BuildConfig.DEBUG) {
    Log.d("ALLSAFE", "Secret input received (length: " + secret.getText().toString().length() + ")");
}
```

#### 2. Build-Time Protection: Strip Logs with R8 / ProGuard

To prevent developer oversight when debug logs are left in code, configure R8 / ProGuard (`proguard-rules.pro`) to automatically remove `Log` class methods from the final release APK:

```proguard
# Remove verbose, debug, and info logs in production release builds
-assumenosideeffects class android.util.Log {
    public static int v(...);
    public static int d(...);
    public static int i(...);
}
```

#### 3. Best Practice: Production-Aware Logging Frameworks (Timber)

Instead of calling `android.util.Log` directly, use a framework like [Timber](https://github.com/JakeWharton/timber). Timber separates logging logic from behavior by allowing you to plant log trees dynamically:

```java
// In Application class:
if (BuildConfig.DEBUG) {
    Timber.plant(new Timber.DebugTree());
}

// In code:
Timber.d("Processing input..."); // Silently dropped in release builds
```

#### 4. verification & Testing (OWASP MASTG)

To verify that logging controls are functioning correctly in production release builds, cross-reference the following test cases:

- [MASTG-TEST-0231: References to Logging APIs - OWASP](https://mas.owasp.org/MASTG/tests/android/MASVS-STORAGE/MASTG-TEST-0231/)
- [MASTG-TEST-0203: Runtime Use of Logging APIs - OWASP](https://mas.owasp.org/MASTG/tests/android/MASVS-STORAGE/MASTG-TEST-0203/)

### Impact

If secrets, access tokens, session identifiers, or personally identifiable information are written to logs, they may become accessible to developers, attackers with debugging access, rooted devices, OEM diagnostic software, or crash reporting services. This can lead to credential disclosure, account compromise, or privacy violations.

Documented cases of log-based incidents:

- [Leaked OAuth response code in logs in Coinbase - hackerone](https://hackerone.com/reports/5314)
- [Logged plaintext passwords in EquityPandit - SentinelOne](https://www.sentinelone.com/vulnerability-database/cve-2019-25605/)

## Module 2: Hardcoded Credentials

Embedding static strings such as API secrets, database passwords, or private keys directly inside application code or resource files relies on "security through obscurity." Android APKs can be effortlessly decompiled back into readable Java/Kotlin code or resource XMLs.

In the OWASP Mobile Application Security framework, embedding secrets inside the application binary falls under [MASWE-0005: API Keys Hardcoded in the App Package](https://mas.owasp.org/MASWE/MASVS-AUTH/MASWE-0005/) and contributes to violations of MASVS-AUTH and MASVS-CRYPTO, depending on the type of secret. It also maps to [CWE-798: Use of Hard-coded Credentials](https://cwe.mitre.org/data/definitions/798.html).

As the screenshot below shows, we need to find two (2) hardcoded username and password combinations.

[screenshot of login screen]

### Dynamic Analysis

As we can see in the screenshot, there's a button to initiate login request, so the first thing we could try is to intercept the request using Burp Suite as a proxy.

[screenshot of login request in burp suite]

```http
POST / HTTP/1.1
Host: dev.infosecadventures.com
Content-Type: application/soap+xml; charset=utf-8
Content-Length: 552
Accept-Encoding: gzip, deflate, br
User-Agent: okhttp/4.9.0
Connection: keep-alive

<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
<soap:Header>
    <UsernameToken xmlns="http://siebel.com/webservices">
        superadmin
    </UsernameToken>
    <PasswordText xmlns="http://siebel.com/webservices">
        supersecurepassword
    </PasswordText>
    <SessionType xmlns="http://siebel.com/webservices">
        None
    </SessionType>
</soap:Header>
<soap:Body>
    <!-- data goes here -->
</soap:Body>
</soap:Envelope>
```

We can clearly see it's a SOAP request for login, where the UsernameToken is **"superadmin"** and the PasswordText is **"supersecurepassword"**, both hardcoded in the application. This confirms that the application is embedding authentication credentials directly into outbound requests. However, to determine whether additional secrets are present elsewhere in the APK, we continue with static analysis.

[screenshot of the login request in burp suite]

To verify there are no other hardcoded credentials, we can use static analysis tools such as JADX-GUI to decompile the APK file and inspect the source code.

### Static Analysis

Using JADX-GUI we can decompile the APK file and inspect the source code of the `HardcodedCredentials` fragment, which is shown below.

[screenshot of HardcodedCredentials fragment in JADX-GUI]

We can see on line 29 that there's a POST request body being built, with the username and password hardcoded in the application. Those credentials match what we saw in Burp Suite, so we have found **the first pair of credentials**: `superadmin:supersecurepassword`.

Now, to find the second pair of credentials, we need to read and understand the source code of the login callback function, which is shown below, in the Code Breakdown.

```java
package infosecadventures.allsafe.challenges;

import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Button;
import androidx.constraintlayout.widget.ConstraintLayout;
import androidx.fragment.app.Fragment;
import androidx.fragment.app.FragmentActivity;
import infosecadventures.allsafe.R;
import infosecadventures.allsafe.utils.SnackUtil;
import java.io.IOException;
import kotlin.Metadata;
import kotlin.jvm.internal.DefaultConstructorMarker;
import kotlin.jvm.internal.Intrinsics;
import okhttp3.Call;
import okhttp3.Callback;
import okhttp3.MediaType;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.RequestBody;
import okhttp3.Response;

/* JADX INFO: compiled from: HardcodedCredentials.kt */
/* JADX INFO: loaded from: classes4.dex */
@Metadata(d1 = {"\u0000&\n\u0002\u0018\u0002\n\u0002\u0018\u0002\n\u0002\b\u0003\n\u0002\u0018\u0002\n\u0000\n\u0002\u0018\u0002\n\u0000\n\u0002\u0018\u0002\n\u0000\n\u0002\u0018\u0002\n\u0002\b\u0002\u0018\u0000 \f2\u00020\u0001:\u0001\fB\u0007¢\u0006\u0004\b\u0002\u0010\u0003J&\u0010\u0004\u001a\u0004\u0018\u00010\u00052\u0006\u0010\u0006\u001a\u00020\u00072\b\u0010\b\u001a\u0004\u0018\u00010\t2\b\u0010\n\u001a\u0004\u0018\u00010\u000bH\u0016¨\u0006\r"}, d2 = {"Linfosecadventures/allsafe/challenges/HardcodedCredentials;", "Landroidx/fragment/app/Fragment;", "<init>", "()V", "onCreateView", "Landroid/view/View;", "inflater", "Landroid/view/LayoutInflater;", "container", "Landroid/view/ViewGroup;", "savedInstanceState", "Landroid/os/Bundle;", "Companion", "app_debug"}, k = 1, mv = {2, 1, 0}, xi = ConstraintLayout.LayoutParams.Table.LAYOUT_CONSTRAINT_VERTICAL_CHAINSTYLE)
public final class HardcodedCredentials extends Fragment {
    public static final String BODY = 
        "<soap:Envelope xmlns:soap=\"http://schemas.xmlsoap.org/soap/envelope/\">\n" +
        "    <soap:Header>\n" +
        "        <UsernameToken xmlns=\"http://siebel.com/webservices\">superadmin</UsernameToken>\n" +
        "        <PasswordText xmlns=\"http://siebel.com/webservices\">supersecurepassword</PasswordText>\n" +
        "        <SessionType xmlns=\"http://siebel.com/webservices\">None</SessionType>\n" +
        "    </soap:Header>\n" +
        "    <soap:Body>\n" +
        "        <!-- data goes here -->\n" +
        "    </soap:Body>\n" +
        "</soap:Envelope>";

    /* JADX INFO: renamed from: Companion, reason: from kotlin metadata */
    public static final Companion INSTANCE = new Companion(null);
    private static final MediaType SOAP = MediaType.INSTANCE.parse("application/soap+xml; charset=utf-8");

    @Override // androidx.fragment.app.Fragment
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        Intrinsics.checkNotNullParameter(inflater, "inflater");
        View view = inflater.inflate(R.layout.fragment_hardcoded_credentials, container, false);
        View viewFindViewById = view.findViewById(R.id.request);
        Intrinsics.checkNotNullExpressionValue(viewFindViewById, "findViewById(...)");
        Button request = (Button) viewFindViewById;
        request.setOnClickListener(new View.OnClickListener() { // from class: infosecadventures.allsafe.challenges.HardcodedCredentials$$ExternalSyntheticLambda0
            @Override // android.view.View.OnClickListener
            public final void onClick(View view2) {
                HardcodedCredentials.onCreateView$lambda$0(this.f$0, view2);
            }
        });
        return view;
    }

    /* JADX INFO: Access modifiers changed from: private */
    public static final void onCreateView$lambda$0(HardcodedCredentials this$0, View it) {
        OkHttpClient client = new OkHttpClient();
        RequestBody body = RequestBody.INSTANCE.create(BODY, SOAP);
        Request.Builder builder = new Request.Builder();
        String string = this$0.getString(R.string.dev_env);
        Intrinsics.checkNotNullExpressionValue(string, "getString(...)");
        Request req = builder.url(string).post(body).build();
        client.newCall(req).enqueue(new Callback() { // from class: infosecadventures.allsafe.challenges.HardcodedCredentials$onCreateView$1$1
            @Override // okhttp3.Callback
            public void onResponse(Call call, Response response) {
                Intrinsics.checkNotNullParameter(call, "call");
                Intrinsics.checkNotNullParameter(response, "response");
            }

            @Override // okhttp3.Callback
            public void onFailure(Call call, IOException e) {
                Intrinsics.checkNotNullParameter(call, "call");
                Intrinsics.checkNotNullParameter(e, "e");
            }
        });
        SnackUtil snackUtil = SnackUtil.INSTANCE;
        FragmentActivity fragmentActivityRequireActivity = this$0.requireActivity();
        Intrinsics.checkNotNullExpressionValue(fragmentActivityRequireActivity, "requireActivity(...)");
        snackUtil.simpleMessage(fragmentActivityRequireActivity, "Under development!");
    }

    /* JADX INFO: compiled from: HardcodedCredentials.kt */
    @Metadata(d1 = {"\u0000\u001a\n\u0002\u0018\u0002\n\u0002\u0010\u0000\n\u0002\b\u0003\n\u0002\u0018\u0002\n\u0002\b\u0003\n\u0002\u0010\u000e\n\u0000\b\u0086\u0003\u0018\u00002\u00020\u0001B\t\b\u0002¢\u0006\u0004\b\u0002\u0010\u0003R\u0013\u0010\u0004\u001a\u0004\u0018\u00010\u0005¢\u0006\b\n\u0000\u001a\u0004\b\u0006\u0010\u0007R\u000e\u0010\b\u001a\u00020\tX\u0086T¢\u0006\u0002\n\u0000¨\u0006\n"}, d2 = {"Linfosecadventures/allsafe/challenges/HardcodedCredentials$Companion;", "", "<init>", "()V", "SOAP", "Lokhttp3/MediaType;", "getSOAP", "()Lokhttp3/MediaType;", "BODY", "", "app_debug"}, k = 1, mv = {2, 1, 0}, xi = ConstraintLayout.LayoutParams.Table.LAYOUT_CONSTRAINT_VERTICAL_CHAINSTYLE)
    public static final class Companion {
        public /* synthetic */ Companion(DefaultConstructorMarker defaultConstructorMarker) {
            this();
        }

        private Companion() {
        }

        public final MediaType getSOAP() {
            return HardcodedCredentials.SOAP;
        }
    }
}
```

#### Code Breakdown

```java
public final class HardcodedCredentials extends Fragment {
    public static final String BODY = 
        "<soap:Envelope xmlns:soap=\"http://schemas.xmlsoap.org/soap/envelope/\">\n" +
        "    <soap:Header>\n" +
        "        <UsernameToken xmlns=\"http://siebel.com/webservices\">superadmin</UsernameToken>\n" +
        "        <PasswordText xmlns=\"http://siebel.com/webservices\">supersecurepassword</PasswordText>\n" +
        "        <SessionType xmlns=\"http://siebel.com/webservices\">None</SessionType>\n" +
        "    </soap:Header>\n" +
        "    <soap:Body>\n" +
        "        <!-- data goes here -->\n" +
        "    </soap:Body>\n" +
        "</soap:Envelope>";
    
    public static final Companion INSTANCE = new Companion(null);
    private static final MediaType SOAP = MediaType.INSTANCE.parse("application/soap+xml; charset=utf-8");

    // onCreateView lifecycle method (called when the fragment is created)
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        // Check if the inflater and container are not null
        Intrinsics.checkNotNullParameter(inflater, "inflater");
        
        // Inflate the layout for this fragment
        View view = inflater.inflate(R.layout.fragment_hardcoded_credentials, container, false);
        
        // Get the request button from the layout and check if it's not null
        View viewFindViewById = view.findViewById(R.id.request);
        Intrinsics.checkNotNullExpressionValue(viewFindViewById, "findViewById(...)");
        Button request = (Button) viewFindViewById;
        
        // Set an onClick listener for the request button
        request.setOnClickListener(new View.OnClickListener() {
            @Override
            public final void onClick(View view2) {
                HardcodedCredentials.onCreateView$lambda$0(this.f$0, view2);
            }
        });

        // Return the view
        return view;
    }

    // onClick method for the request button
    public static final void onCreateView$lambda$0(HardcodedCredentials this$0, View it) {
        // Create a new OkHttpClient
        OkHttpClient client = new OkHttpClient();

        // Create a RequestBody from the BODY string and SOAP media type
        RequestBody body = RequestBody.INSTANCE.create(BODY, SOAP);

        // Create a new Request.Builder
        Request.Builder builder = new Request.Builder();

        // Get the dev_env string from the resources and check if it's not null
        String string = this$0.getString(R.string.dev_env);
        Intrinsics.checkNotNullExpressionValue(string, "getString(...)");
        
        // Build the request with the dev_env URL and the request body
        Request req = builder.url(string).post(body).build();

        // Enqueue the request (execute asynchronously)
        client.newCall(req).enqueue(new Callback() {
            // In case of success, do nothing
            @Override
            public void onResponse(Call call, Response response) {
                Intrinsics.checkNotNullParameter(call, "call");
                Intrinsics.checkNotNullParameter(response, "response");
            }

            // In case of failure, do nothing
            @Override
            public void onFailure(Call call, IOException e) {
                Intrinsics.checkNotNullParameter(call, "call");
                Intrinsics.checkNotNullParameter(e, "e");
            }
        });
        
        // Displays a Toast message with the string "Under development!"
        SnackUtil snackUtil = SnackUtil.INSTANCE;
        FragmentActivity fragmentActivityRequireActivity = this$0.requireActivity();
        Intrinsics.checkNotNullExpressionValue(fragmentActivityRequireActivity, "requireActivity(...)");
        snackUtil.simpleMessage(fragmentActivityRequireActivity, "Under development!");
    }

    public static final class Companion {
        public final MediaType getSOAP() {
            return HardcodedCredentials.SOAP;
        }
    }
}
```

In Android applications, hardcoded credentials or sensitive strings aren't always placed directly as raw code variables. They are frequently stored in application resource files like `res/values/strings.xml`.

If you look closely at lines 56-57 of the decompiled fragment (without the comments), you will see:

```java
String string = this$0.getString(R.string.dev_env);
Intrinsics.checkNotNullExpressionValue(string, "getString(...)");
```

The application is fetching a string resource referenced by `R.string.dev_env`. Now we just need to find the `strings.xml` and search for the value of `dev_env`.

```xml
<resources>
    <string name="dev_env">https://admin:password123@dev.infosecadventures.com</string>
</resources>
```

[screenshot of strings.xml]

We can see that the value of `dev_env` is `https://admin:password123@dev.infosecadventures.com`. This is the second hardcoded credential we needed to find.

### Remediation & Code Fix

One of the most common misconceptions in mobile development is that APK files are "compiled" and therefore hide implementation details. In reality, Android applications can be decompiled with freely available tools such as [JADX](https://github.com/skylot/jadx) or [Apktool](https://apktool.org/), allowing attackers to recover embedded constants, strings, resources, and configuration files. Consequently, any credential shipped inside the application should be considered public.

### Impact

Hardcoded credentials should be considered compromised the moment an application is distributed. Unlike server-side secrets, client-side secrets cannot be revoked simply by hiding the source code, since every user receives a complete copy of the APK.

Depending on the credential, an attacker may be able to:

- Authenticate as privileged users.
- Access internal APIs or development environments.
- Extract additional sensitive information from backend services.
- Reuse credentials across staging and production environments.
- Reverse engineer proprietary business logic.
- Impersonate legitimate clients by replaying authenticated requests.

In real-world applications, exposed API keys have resulted in unauthorized access to cloud resources, excessive billing charges, data leakage, and complete backend compromise. Even credentials intended only for development environments can become valuable if those environments contain production data or share authentication infrastructure.

There are some documented cases of hardcoded credentials in real-world applications:

- [Twilio Credentials Hardcoded in Mobile Apps Expose Calls, Texts - Eduard Kovacs, SecurityWeek](https://www.securityweek.com/twilio-credentials-hardcoded-mobile-apps-expose-calls-texts/)
- [Many Mobile Apps Unnecessarily Leak Hardcoded Keys: Analysis - Ionut Arghire, SecurityWeek](https://www.securityweek.com/many-mobile-apps-unnecessarily-leak-hardcoded-keys-analysis/)

## References

- [Logcat - Android Developers](https://developer.android.com/tools/logcat)
- [Log Info Disclosure - Android Developers](https://developer.android.com/privacy-and-security/risks/log-info-disclosure)
- [Additional Proguard Rules - Android Developers](https://developer.android.com/topic/performance/app-optimization/additional-rule-types)
- [How to Remove Debug Logging with ProGuard? - Guardsquare](https://www.guardsquare.com/blog/how-to-remove-debug-logging-with-proguard-guardsquare)
- [M9: Insecure Data Storage - OWASP Mobile Top 10](https://owasp.org/www-project-mobile-top-10/2023-risks/m9-insecure-data-storage.html)
- [CWE-532: Insertion of Sensitive Information into Log File - CWE](https://cwe.mitre.org/data/definitions/532.html)
- [MASTG-TEST-0231: References to Logging APIs - OWASP](https://mas.owasp.org/MASTG/tests/android/MASVS-STORAGE/MASTG-TEST-0231/)
- [MASTG-TEST-0203: Runtime Use of Logging APIs - OWASP](https://mas.owasp.org/MASTG/tests/android/MASVS-STORAGE/MASTG-TEST-0203/)
- [MASWE-0005: API Keys Hardcoded in the App Package - OWASP](https://mas.owasp.org/MASWE/MASVS-AUTH/MASWE-0005/)
- [CWE-798: Use of Hard-coded Credentials - CWE](https://cwe.mitre.org/data/definitions/798.html)
- [Twilio Credentials Hardcoded in Mobile Apps Expose Calls, Texts - Eduard Kovacs, SecurityWeek](https://www.securityweek.com/twilio-credentials-hardcoded-mobile-apps-expose-calls-texts/)
- [Many Mobile Apps Unnecessarily Leak Hardcoded Keys: Analysis - Ionut Arghire, SecurityWeek](https://www.securityweek.com/many-mobile-apps-unnecessarily-leak-hardcoded-keys-analysis/)

