# Chapter 2: The Tentacles (API Integrations)

A brain (Dashboard) is useless without hands. To manipulate the world of social media, we needed tentacles—direct API connections to the major platforms. This was the most grueling part of the project, as every platform has its own gatekeepers and distinct authentication rituals.

## 1. X (Twitter): The Volatile Veteran
X's API has changed drastically. We utilized the `Tweepy` library (and later raw requests) to handle OAuth 1.0a for posting.
*   **The Challenge**: Managing media uploads. You cannot just "post an image"; you must first upload the media to an endpoint, get a `media_id`, wait for processing, and *then* attach it to a tweet.
*   **The Solution**: We built a robust `upload_media()` function that handles the async nature of X's media processing.

## 2. Threads: The New Frontier
Integrating Threads was a journey through the Meta for Developers portal. Unlike X, Threads requires a "Meta App" and a strict "Tester" hierarchy.
*   **The Setup**:
    1.  Create a Meta App.
    2.  Add the "Threads" product.
    3.  Add the target Instagram/Threads account as a "Tester".
    4.  Accept the invite on the mobile app.
    5.  Generate the `THREADS_USER_ID` and long-lived `THREADS_ACCESS_TOKEN`.
*   **The Wins**: Once connected, the API is surprisingly stable. We can push text and links seamlessly.

## 3. Pinterest: The Walled Garden
Pinterest was the hardest nut to crack. Their API team is strict.
*   **The Rejection**: Our initial application was rejected. Reason? "Lack of demo integration." They demand a video showing exactly how the app uses the API.
*   **The Pivot**: We had to record a walkthrough of our script running, submitting it to the review team to prove we weren't spamming.
*   **The Code**: We focused on Board management, ensuring every pin lands in a relevant, SEO-optimized board.

## 4. YouTube: The Heavy Lifter
For YouTube, we used the Google Data API.
*   **The Goal**: Not just uploading videos, but automating the *metadata*. Our script generates SEO-friendly titles and descriptions based on the video content before pushing it live.
*   **The Quota**: Google has strict daily quotas. We had to optimize our calls to avoid hitting the ceiling during testing.

## Summary
Each API is a separate fiefdom. By abstracting them into a single Python interface (`poster.post_to_all(content)`), we effectively created a "Universal Remote" for social media.
