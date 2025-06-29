# 🎥 AI_ReelForge - n8n Workflow to generate videos with Google Veo3 and upload it on Youtube

This `n8n` workflow automates the process of **creating AI-generated videos with Google Veo3**, uploading the final video to **Google Drive and YouTube**, and **logging everything in Google Sheets**.

---

## ⚙️ Prerequisites

1. An [n8n](https://n8n.io/) environment.
2. Access to:
   - [Google Sheets](https://docs.google.com/spreadsheets/)
   - [Fal.ai](https://fal.ai/) API key
   - [UploadPost](https://app.upload-post.com/) account for uploading videos to YouTube
   - [Google Drive](https://drive.google.com) and [OpenAI](https://openai.com) credentials

---

## 📝 Step-by-Step Setup Guide (All Node Instructions)

---

### 🟢 1. Manual Trigger / Schedule Trigger
- **Node**: `When clicking ‘Test workflow’` or `Schedule Trigger`
- **Purpose**: Start the flow manually or on schedule (recommended: every 5 mins)
- **Setup**: No configuration needed.

---

### 📄 2. Google Sheet (Video Instructions)
- **Node**: `Get new video`
- **Setup**:
  - Create [this sample Google Sheet](https://docs.google.com/spreadsheets/d/1pcoY9N_vQp44NtSRR5eskkL5Qd0N0BGq7Jh_4m-7VEQ/edit?usp=sharing).
  - Fill columns:
    - `PROMPT`: A short description of the video idea
    - `DURATION`: Length of the video
  - Leave `VIDEO` and `YOUTUBE_URL` blank — they'll be auto-filled.
  - Connect your **Google Sheets OAuth** credentials in this node.

---

### 🧠 3. Set data
- **Node**: `Set data`
- **Purpose**: Formats the prompt using the Google Sheet input
- **Setup**: No changes needed.

---

### 🎬 4. Create Video with Veo3
- **Node**: `Create Video` (HTTP Request)
- **Setup**:
  - Method: `POST`
  - URL: `https://queue.fal.run/fal-ai/veo3`
  - Auth: `HTTP Header Auth`
    - Header Name: `Authorization`
    - Header Value: `Key YOUR_FAL_API_KEY`
  - Body:
    ```json
    {
      "prompt": "{{$json.prompt}}"
    }
    ```

---

### ⏳ 5. Wait
- **Node**: `Wait 60 sec.`
- **Purpose**: Delay to let video generation complete
- **Setup**: No changes needed unless you want shorter wait time.

---

### 📡 6. Get Video Generation Status
- **Node**: `Get status` (HTTP Request)
- **URL**: `https://queue.fal.run/fal-ai/veo3/requests/{{ $('Create Video').item.json.request_id }}/status`
- **Auth**: Same Fal API Header as above

---

### ✅ 7. Check Completion
- **Node**: `Completed?` (If condition)
- **Setup**:
  - Condition: `$json.status == "COMPLETED"`

---

### 🔗 8. Get Video URL
- **Node**: `Get Url Video` (HTTP Request)
- **URL**: `https://queue.fal.run/fal-ai/veo3/requests/{{ $json.request_id }}`

---

### ✍️ 9. Generate YouTube Title
- **Node**: `Generate title` (OpenAI)
- **Prompt Setup**: Uses `PROMPT` as the input, and GPT-4o to generate a catchy title.
- **Credential**: Connect your OpenAI key.

---

### 📥 10. Download the Video
- **Node**: `Get File Video`
- **URL**: `{{$('Get Url Video').item.json.video.url}}`
- **Setup**: No changes needed.

---

### ☁️ 11. Upload to Google Drive
- **Node**: `Upload Video` (Google Drive)
- **Setup**:
  - Credential: Connect Google Drive
  - Folder ID: Choose your destination folder
  - File name: `{{ $now.format('yyyyLLddHHmmss') }}-{{ $('Get Url Video').item.json.video.file_name }}`

---

### 🧠 12. Upload to YouTube via UploadPost API
- **Node**: `HTTP Request` to `https://api.upload-post.com/api/upload`
- **Setup**:
  - Auth Header:
    - Name: `Authorization`
    - Value: `Apikey YOUR_UPLOADPOST_API_KEY`
  - Fields:
    - `title`: `={{ $('Generate title').item.json.message.content }}`
    - `user`: Replace `YOUR_USERNAME` with your UploadPost username
    - `platform[]`: `youtube`
    - `video`: `formBinaryData` (link to previous video download)

---

### 📝 13. Update Google Sheet with Video URL
- **Node**: `Update result`
- **Setup**:
  - Credential: Connect Google Sheets OAuth
  - Document ID: Match original sheet
  - Update `VIDEO` column with `{{ $('Get Url Video').item.json.video.url }}`

---

### 📝 14. Update Google Sheet with YouTube URL
- **Node**: `Update Youtube URL`
- **Setup**:
  - Credential: Same Google Sheet credential
  - Updates `YOUTUBE_URL` with: `https://youtu.be/{{ $json.results.youtube.video_id }}`

---

## 🚀 Final Notes

- ✅ Make sure all credentials are connected inside `n8n`
- ✅ Always test manually first before scheduling
- ✅ API rate limits may apply on Fal.ai and UploadPost

---

## 📊 Suggested Workflow Triggers

| Trigger Type      | Use Case                      |
|------------------|-------------------------------|
| Manual Trigger   | Testing the flow manually      |
| Schedule Trigger | Run every 5–10 minutes         |

---

