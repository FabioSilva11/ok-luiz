# ok-luiz

## Introdução

O `VoiceRecognitionService` é um serviço Android que reconhece o comando de voz "ok Luiz" e exibe um Toast como resposta. Este serviço funciona mesmo quando o aplicativo está fechado.

## Pré-requisitos

- Android Studio ou qualquer IDE compatível com Android.
- Conhecimento básico de desenvolvimento Android e Java.

## Configuração do Projeto

### Passo 1: Configurar o `build.gradle`

Adicione as seguintes dependências no arquivo `build.gradle` do seu módulo (geralmente `app/build.gradle`):

```gradle
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'androidx.appcompat:appcompat:1.4.0'
    implementation 'com.google.android.material:material:1.6.1'
    implementation 'com.google.android.gms:play-services-ads:20.1.0'
    implementation 'com.google.code.gson:gson:2.8.7'
}
```

### Passo 2: Criar o Serviço de Reconhecimento de Voz

Crie um novo arquivo Java chamado `VoiceRecognitionService.java` e adicione o seguinte código:

```java
import android.app.Notification;
import android.app.NotificationChannel;
import android.app.NotificationManager;
import android.app.Service;
import android.content.Intent;
import android.os.Build;
import android.os.Bundle;
import android.os.IBinder;
import android.speech.RecognitionListener;
import android.speech.RecognizerIntent;
import android.speech.SpeechRecognizer;
import android.widget.Toast;

import java.util.ArrayList;

public class VoiceRecognitionService extends Service {

    private static final String CHANNEL_ID = "VoiceRecognitionServiceChannel";
    private SpeechRecognizer speechRecognizer;

    @Override
    public void onCreate() {
        super.onCreate();
        createNotificationChannel();
        startForeground(1, getNotification());

        speechRecognizer = SpeechRecognizer.createSpeechRecognizer(this);
        speechRecognizer.setRecognitionListener(new RecognitionListener() {
            @Override
            public void onReadyForSpeech(Bundle params) {}

            @Override
            public void onBeginningOfSpeech() {}

            @Override
            public void onRmsChanged(float rmsdB) {}

            @Override
            public void onBufferReceived(byte[] buffer) {}

            @Override
            public void onEndOfSpeech() {}

            @Override
            public void onError(int error) {
                restartListening();
            }

            @Override
            public void onResults(Bundle results) {
                ArrayList<String> matches = results.getStringArrayList(SpeechRecognizer.RESULTS_RECOGNITION);
                if (matches != null && !matches.isEmpty()) {
                    String command = matches.get(0);
                    if (command.equalsIgnoreCase("ok Luiz")) {
                        Toast.makeText(VoiceRecognitionService.this, "Comando reconhecido!", Toast.LENGTH_LONG).show();
                    }
                }
                restartListening();
            }

            @Override
            public void onPartialResults(Bundle partialResults) {}

            @Override
            public void onEvent(int eventType, Bundle params) {}
        });

        restartListening();
    }

    private void restartListening() {
        Intent intent = new Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH);
        intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL, RecognizerIntent.LANGUAGE_MODEL_FREE_FORM);
        intent.putExtra(RecognizerIntent.EXTRA_CALLING_PACKAGE, this.getPackageName());
        intent.putExtra(RecognizerIntent.EXTRA_SPEECH_INPUT_COMPLETE_SILENCE_LENGTH_MILLIS, new Long(3000));
        intent.putExtra(RecognizerIntent.EXTRA_SPEECH_INPUT_MINIMUM_LENGTH_MILLIS, new Long(3000));
        intent.putExtra(RecognizerIntent.EXTRA_PREFER_OFFLINE, true);
        speechRecognizer.startListening(intent);
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        return START_STICKY;
    }

    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        if (speechRecognizer != null) {
            speechRecognizer.destroy();
        }
    }

    private Notification getNotification() {
        Notification.Builder builder = new Notification.Builder(this)
                .setContentTitle("Voice Recognition Service")
                .setContentText("Listening for 'ok Luiz' command")
                .setSmallIcon(R.drawable.ic_launcher_foreground);

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            builder.setChannelId(CHANNEL_ID);
        }

        return builder.build();
    }

    private void createNotificationChannel() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            NotificationChannel serviceChannel = new NotificationChannel(
                    CHANNEL_ID,
                    "Voice Recognition Service Channel",
                    NotificationManager.IMPORTANCE_DEFAULT
            );
            NotificationManager manager = getSystemService(NotificationManager.class);
            if (manager != null) {
                manager.createNotificationChannel(serviceChannel);
            }
        }
    }
}
```

### Passo 3: Adicionar Permissões no Manifesto

No arquivo `AndroidManifest.xml`, adicione as permissões necessárias e declare o serviço:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.voicerecognitionservice">

    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">

        <service
            android:name=".VoiceRecognitionService"
            android:enabled="true"
            android:exported="false" />
    </application>

</manifest>
```

### Passo 4: Iniciar o Serviço

Para iniciar o serviço de reconhecimento de voz, adicione o seguinte código em uma Activity ou outra parte do seu aplicativo:

```java
import android.content.Intent;
import android.os.Bundle;

import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Intent serviceIntent = new Intent(this, VoiceRecognitionService.class);
        startService(serviceIntent);
    }
}
```

### Passo 5: Adicionar Ícone de Notificação

Verifique se você tem um ícone de notificação disponível (`ic_launcher_foreground`). Caso contrário, substitua `R.drawable.ic_launcher_foreground` por um ícone existente em seu projeto ou adicione um novo ícone em `res/drawable`.

## Conclusão

Agora você tem um serviço de reconhecimento de voz configurado que pode reconhecer o comando "ok Luiz" e exibir um Toast como resposta. Este serviço continuará funcionando em segundo plano mesmo quando o aplicativo estiver fechado.
