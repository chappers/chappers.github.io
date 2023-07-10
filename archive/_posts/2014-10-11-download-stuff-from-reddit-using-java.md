---
layout: post
category: web micro log
tags: [java]
---

Previously I've looked at how to download stuff from Reddit. So I thought I'll re-apply the same code, but using Java. Here is the same code to do just that.

```java
import java.io.*;
import java.net.URL;
import java.nio.channels.Channels;
import java.nio.channels.ReadableByteChannel;
import java.nio.charset.Charset;
import java.nio.file.*;
import java.util.Arrays;

import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

public class getWallpaper {
    private static final String OPEN_SUBREDDIT = "http://www.reddit.com/r/%s/.json";

    public static String readAll(Reader rd) throws IOException {
        StringBuilder sb = new StringBuilder();
        int cp;
        while ((cp = rd.read()) != -1) {
            sb.append((char) cp);
        }
        return sb.toString();
    }

    public static JSONObject subredditJson(String subreddit) throws IOException, JSONException {
        InputStream is = new URL(String.format(OPEN_SUBREDDIT, subreddit)).openStream();
        try {
            BufferedReader rd = new BufferedReader(new InputStreamReader(is, Charset.forName("UTF-8")));
            String jsonText = readAll(rd);
            JSONObject json = new JSONObject(jsonText);
            return json;
        } finally {
            is.close();
        }
    }

    static void wallpaperLinks(JSONObject json) {
        String[] wallpapers;
        JSONArray data = json.getJSONObject("data").getJSONArray("children");
        // the children is where the information is contained

        for (int i = 0; i < data.length(); i++) {
            // iterate over JSONArray, and extract url
            JSONObject urlInfo = data.getJSONObject(i);
            String wallpaper = urlInfo.getJSONObject("data").getString("url");
            if (wallpaper.substring(wallpaper.length() - 4, wallpaper.length()).equals(".jpg")) {
                String[] fileName = wallpaper.split("/");
                try {
                    downloadFile(wallpaper, String.format("%s.jpg", fileName[fileName.length - 1]));
                } catch (IOException e) {
                    System.out.println("File not found");
                }
            }
        }
    }

    static void downloadFile(String url, String target) throws IOException {
        URL urlFile = new URL(url);
        ReadableByteChannel rbc = Channels.newChannel(urlFile.openStream());
        FileOutputStream fos = new FileOutputStream(target);
        fos.getChannel().transferFrom(rbc, 0, Long.MAX_VALUE);
    }

    public static void main(String[] args) throws IOException, JSONException {
        JSONObject json = subredditJson("EarthPorn");
        wallpaperLinks(json);
    }
}
```

The first two methods `subredditJson` and `readAll` are for grabbing the required `json` file from Reddit.

The next two methods `wallpaperLinks` and `downloadFile` are for extracting the particular image links and downloading the required information using Java.

What made this project interesting was learning about different libraries for JSON, in particular, the JSON library used in Android is Android specific. This is also roughly three times as much code as the Python implementation, demonstrating to me at least, the beauty that is in Python (at least compared to Java).
