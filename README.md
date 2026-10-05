package com.jarvis.ai;

import android.Manifest;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.os.Bundle;
import android.speech.RecognitionListener;
import android.speech.RecognizerIntent;
import android.speech.SpeechRecognizer;
import android.speech.tts.TextToSpeech;
import android.widget.Button;
import android.widget.TextView;

import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import java.util.ArrayList;
import java.util.Locale;

public class MainActivity extends AppCompatActivity {

    private TextView tvStatus;
    private Button btnAction;

    private SpeechRecognizer speechRecognizer;
    private Intent speechIntent;
    private TextToSpeech textToSpeech;

    private static final int RECORD_AUDIO_REQUEST = 1001;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        tvStatus = findViewById(R.id.tvStatus);
        btnAction = findViewById(R.id.btnAction);

        // Text to Speech
        textToSpeech = new TextToSpeech(this, status -> {
            if (status == TextToSpeech.SUCCESS) {
                textToSpeech.setLanguage(Locale.US);
            }
        });

        // Speech Recognizer
        if (SpeechRecognizer.isRecognitionAvailable(this)) {

            speechRecognizer = SpeechRecognizer.createSpeechRecognizer(this);

            speechIntent = new Intent(
                    RecognizerIntent.ACTION_RECOGNIZE_SPEECH
            );

            speechIntent.putExtra(
                    RecognizerIntent.EXTRA_LANGUAGE_MODEL,
                    RecognizerIntent.LANGUAGE_MODEL_FREE_FORM
            );

            speechIntent.putExtra(
                    RecognizerIntent.EXTRA_LANGUAGE,
                    Locale.getDefault()
            );

            speechIntent.putExtra(
                    RecognizerIntent.EXTRA_MAX_RESULTS,
                    1
            );

            speechRecognizer.setRecognitionListener(
                    new RecognitionListener() {

                        @Override
                        public void onReadyForSpeech(Bundle params) {
                            tvStatus.setText("JARVIS is Listening...");
                        }

                        @Override
                        public void onBeginningOfSpeech() {
                            tvStatus.setText("I'm listening...");
                        }

                        @Override
                        public void onRmsChanged(float rmsdB) {
                        }

                        @Override
                        public void onBufferReceived(byte[] buffer) {
                        }

                        @Override
                        public void onEndOfSpeech() {
                            tvStatus.setText("Processing...");
                        }

                        @Override
                        public void onError(int error) {
                            tvStatus.setText("Ready. Tap to speak again.");
                        }

                        @Override
                        public void onResults(Bundle results) {

                            ArrayList<String> matches =
                                    results.getStringArrayList(
                                            SpeechRecognizer.RESULTS_RECOGNITION
                                    );

                            if (matches != null && !matches.isEmpty()) {

                                String command = matches.get(0);

                                tvStatus.setText(
                                        "You: " + command
                                );

                                respondToCommand(command);
                            }
                        }

                        @Override
                        public void onPartialResults(Bundle partialResults) {
                        }

                        @Override
                        public void onEvent(int eventType, Bundle params) {
                        }
                    }
            );

        } else {
            tvStatus.setText(
                    "Speech recognition is not available"
            );
            btnAction.setEnabled(false);
        }

        btnAction.setOnClickListener(v -> startListening());
    }

    private void startListening() {

        if (ContextCompat.checkSelfPermission(
                this,
                Manifest.permission.RECORD_AUDIO
        ) != PackageManager.PERMISSION_GRANTED) {

            ActivityCompat.requestPermissions(
                    this,
                    new String[]{Manifest.permission.RECORD_AUDIO},
                    RECORD_AUDIO_REQUEST
            );

            return;
        }

        tvStatus.setText("Starting JARVIS...");

        speechRecognizer.startListening(speechIntent);
    }

    private void respondToCommand(String command) {

        String lowerCommand = command.toLowerCase(Locale.ROOT);

        String response;

        if (lowerCommand.contains("hello")
                || lowerCommand.contains("hi")
                || lowerCommand.contains("হ্যালো")) {

            response = "Hello. I am JARVIS. How can I help you?";

        } else if (lowerCommand.contains("who are you")
                || lowerCommand.contains("তুমি কে")) {

            response = "I am JARVIS, your personal AI assistant.";

        } else if (lowerCommand.contains("how are you")
                || lowerCommand.contains("কেমন আছো")) {

            response = "I am ready and listening.";

        } else if (lowerCommand.contains("stop")
                || lowerCommand.contains("বন্ধ")) {

            response = "Okay. I will stop listening.";

        } else {

            response = "I heard you say: " + command;
        }

        tvStatus.setText(response);
        speak(response);
    }

    private void speak(String text) {

        if (textToSpeech != null) {
            textToSpeech.speak(
                    text,
                    TextToSpeech.QUEUE_FLUSH,
                    null,
                    "JARVIS_RESPONSE"
            );
        }
    }

    @Override
    public void onRequestPermissionsResult(
            int requestCode,
            String[] permissions,
            int[] grantResults
    ) {
        super.onRequestPermissionsResult(
                requestCode,
                permissions,
                grantResults
        );

        if (requestCode == RECORD_AUDIO_REQUEST) {

            if (grantResults.length > 0
                    && grantResults[0]
                    == PackageManager.PERMISSION_GRANTED) {

                startListening();

            } else {

                tvStatus.setText(
                        "Microphone permission is required."
                );
            }
        }
    }

    @Override
    protected void onDestroy() {

        if (speechRecognizer != null) {
            speechRecognizer.destroy();
        }

        if (textToSpeech != null) {
            textToSpeech.stop();
            textToSpeech.shutdown();
        }

        super.onDestroy();
    }
}
