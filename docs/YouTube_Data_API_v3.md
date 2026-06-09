Wans, cara paling aman: **pakai YouTube Data API v3**, bukan scraping **HTML (HyperText Markup Language)**.

• YouTube menyediakan endpoint `commentThreads.list` untuk mengambil komentar berdasarkan `videoId`. Endpoint ini bisa mengambil komentar utama, memakai `maxResults` 1–100, dan memakai `pageToken` untuk lanjut ke halaman berikutnya. ([Google for Developers][1])

• Untuk mengambil semua balasan komentar, pakai endpoint `comments.list` dengan `parentId`, karena balasan komentar ditarik dari ID komentar utama. ([Google for Developers][2])

• API ini memakai sistem kuota, jadi script perlu pagination dan pembatasan request. ([Google for Developers][3])

Langkah:

• Buat project di Google Cloud Console.
• Aktifkan **YouTube Data API v3**.
• Buat **API key**.
• Ambil `videoId` dari URL YouTube.
Contoh:
`https://www.youtube.com/watch?v=ABC123xyz`
`videoId = ABC123xyz`

Install dependency:

```bash
pip install requests
```

Script Python:

```python
import csv
import requests
from datetime import datetime

API_KEY = "ISI_API_KEY_KAMU"
VIDEO_ID = "ISI_VIDEO_ID"
OUTPUT_FILE = "youtube_comments.csv"

BASE_URL = "https://www.googleapis.com/youtube/v3/commentThreads"


def get_comments(video_id, api_key, max_pages=None):
    comments = []
    page_token = None
    page_count = 0

    while True:
        params = {
            "part": "snippet",
            "videoId": video_id,
            "key": api_key,
            "maxResults": 100,
            "textFormat": "plainText",
            "order": "time"
        }

        if page_token:
            params["pageToken"] = page_token

        response = requests.get(BASE_URL, params=params, timeout=20)

        if response.status_code != 200:
            raise Exception(f"API error {response.status_code}: {response.text}")

        data = response.json()

        for item in data.get("items", []):
            snippet = item["snippet"]["topLevelComment"]["snippet"]

            comments.append({
                "comment_id": item["snippet"]["topLevelComment"]["id"],
                "author": snippet.get("authorDisplayName", ""),
                "text": snippet.get("textDisplay", ""),
                "like_count": snippet.get("likeCount", 0),
                "published_at": snippet.get("publishedAt", ""),
                "updated_at": snippet.get("updatedAt", "")
            })

        page_count += 1

        if max_pages and page_count >= max_pages:
            break

        page_token = data.get("nextPageToken")

        if not page_token:
            break

    return comments


def save_to_csv(comments, filename):
    fieldnames = [
        "comment_id",
        "author",
        "text",
        "like_count",
        "published_at",
        "updated_at"
    ]

    with open(filename, "w", newline="", encoding="utf-8-sig") as file:
        writer = csv.DictWriter(file, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(comments)


if __name__ == "__main__":
    result = get_comments(VIDEO_ID, API_KEY)
    save_to_csv(result, OUTPUT_FILE)

    print(f"Selesai. Total komentar: {len(result)}")
    print(f"File tersimpan: {OUTPUT_FILE}")
```

Output CSV:

| Kolom          | Isi                   |
| -------------- | --------------------- |
| `comment_id`   | ID komentar           |
| `author`       | Nama akun komentator  |
| `text`         | Isi komentar          |
| `like_count`   | Jumlah like komentar  |
| `published_at` | Waktu komentar dibuat |
| `updated_at`   | Waktu komentar diedit |

Catatan penting:

• Video dengan komentar nonaktif tidak bisa diambil.
• Komentar yang dihapus, disembunyikan, atau dibatasi tidak selalu muncul.
• Jangan scrape HTML YouTube langsung. Pakai API resmi agar lebih stabil dan aman dari sisi aturan platform.

[1]: https://developers.google.com/youtube/v3/docs/commentThreads/list?utm_source=chatgpt.com "CommentThreads: list | YouTube Data API"
[2]: https://developers.google.com/youtube/v3/docs/comments/list "Comments: list  |  YouTube Data API  |  Google for Developers"
[3]: https://developers.google.com/youtube/v3/guides/quota_and_compliance_audits?utm_source=chatgpt.com "Quota and Compliance Audits | YouTube Data API"
