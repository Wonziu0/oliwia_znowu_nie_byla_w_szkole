package com.example.url;

import android.os.Bundle;
import android.os.AsyncTask;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;
import androidx.appcompat.app.AppCompatActivity;
import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;

public class MainActivity extends AppCompatActivity {
    private static final String TAG = "MainActivity";
    private Button btn1;
    private TextView txt;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        btn1 = findViewById(R.id.button);
        txt = findViewById(R.id.textView);

        btn1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                new DownloadTask().execute("http://json.itmargen.com/5TP/");
            }
        });
    }

    private class DownloadTask extends AsyncTask<String, Void, String> {
        @Override
        protected String doInBackground(String... urls) {
            try {
                return downloadData(urls[0]);
            } catch (Exception e) {
                Log.e(TAG, "Error downloading data: " + e.getMessage(), e);
                return null;
            }
        }

        @Override
        protected void onPostExecute(String result) {
            if (result != null) {
                String formattedJson = formatJsonForReadability(result);
                txt.setText(formattedJson);
            } else {
                txt.setText("Error fetching data");
            }
        }
    }

    public String downloadData(String urlString) throws Exception {
        Log.d(TAG, "Attempting to download data from URL: " + urlString);
        URL url = new URL(urlString);
        HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
        urlConnection.setRequestMethod("GET");
        urlConnection.setReadTimeout(10000);
        urlConnection.setConnectTimeout(15000);
        try {
            int responseCode = urlConnection.getResponseCode();
            Log.d(TAG, "Server response code: " + responseCode);
            if (responseCode == HttpURLConnection.HTTP_OK) {
                BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(urlConnection.getInputStream()));
                StringBuilder result = new StringBuilder();
                String line;
                while ((line = bufferedReader.readLine()) != null) {
                    result.append(line);
                }
                bufferedReader.close();
                Log.d(TAG, "Data downloaded successfully");
                return result.toString();
            } else {
                Log.e(TAG, "Server returned error response code: " + responseCode);
                return null;
            }
        } finally {
            urlConnection.disconnect();
        }
    }

    private String formatJsonForReadability(String jsonString) {
        StringBuilder formattedString = new StringBuilder();
        try {
            JSONArray jsonArray = new JSONArray(jsonString);
            for (int i = 0; i < jsonArray.length(); i++) {
                JSONObject jsonObject = jsonArray.getJSONObject(i);
                formattedString.append("Tytuł: ").append(jsonObject.optString("title", "N/A")).append("\n");
                formattedString.append("Opis: ").append(jsonObject.optString("description", "N/A")).append("\n");
                formattedString.append("Data: ").append(jsonObject.optString("date", "N/A")).append("\n");
                formattedString.append("Autor: ").append(jsonObject.optString("author", "N/A")).append("\n");
                formattedString.append("Zawartość: ").append(jsonObject.optString("content", "N/A")).append("\n");
                formattedString.append("\n");
            }
        } catch (JSONException e) {
            Log.e(TAG, "Error formatting JSON: " + e.getMessage(), e);
            return "Error formatting data";
        }
        return formattedString.toString().trim();
    }
}
