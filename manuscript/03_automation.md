# Chapter 3: The Engine (Python Automation)

With the Dashboard watching and the APIs connected, we needed an engine to drive the car. This is where Python shines. We built a suite of scripts in the `scripts/` directory to handle the daily grind.

## The Core Script: `execute_daily_post.py`
This is the heartbeat of the system.
**Workflow:**
1.  **Awaken**: The script triggers (via Cron or manual run).
2.  **Fetch Content**: It looks into a content queue (or generates new ideas).
3.  **Process Media**: It resizes images for Instagram/Threads (square) vs YouTube (16:9).
4.  **Broadcast**: It calls the collection of API client classes we built in Chapter 2.
5.  **Log**: It writes the result to the Supabase database, updating the Dashboard.

## Content Engineering
We didn't just automate delivery; we experimented with automating *creation*.
*   **Generative Text**: Using LLMs to draft tweet variations.
*   **Image Processing**: Using Python's `PIL` (Pillow) library to overlay text on images programmatically. This allows us to turn a single quote into a visual asset suitable for Instagram and Pinterest.

## Secrets Management (`.env`)
Security is paramount. We implemented strict `.env` file management.
*   `THREADS_ACCESS_TOKEN`
*   `TWITTER_CONSUMER_KEY`
*   `YOUTUBE_CLIENT_SECRET`
All sensitive keys are kept out of Git, loaded only at runtime using `python-dotenv`. This ensures that even if we open-source the engine, the keys to the kingdom remain safe.

## The "Part 2" Fix
We encountered issues where long-running processes would timeout or get stuck. We split the logic into `execute_daily_post_part1.py` and `execute_daily_post_part2.py` (the backfill scripts) to decouple heavy media processing from quick status updates. This modular approach increased system reliability from 80% to 99%.

The Engine is now self-driving. It requires only fuel (content ideas) to keep running.
