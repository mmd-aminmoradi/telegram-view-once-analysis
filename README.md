```markdown
# telegram-view-once-analysis
Telegram View Once Media Is Not Truly Ephemeral

**Author: Amin Moradi (amin_moradi)**  
**Telegram Channel: [https://t.me/Hezar_code](https://t.me/Hezar_code)**

---

## Table of Contents

1. What is View Once?
2. Executive Summary
3. Motivation for This Research
4. Methodology and Test Environment
5. Step-by-Step Reproduction Steps
6. Official Telegram Response
7. Critical Analysis of Telegram's Response
8. Technical Deep Dive: No Screenshot, No Opening
9. Protocol-Level Vulnerability Explanation
10. Why Telegram Won't Fix This (Analysis)
11. Security Recommendations
12. Frequently Asked Questions (FAQ)
13. About the Author
14. Changelog
15. Acknowledgments
16. References and Links

---

## 1. What is View Once?

Telegram has a feature called **View Once** (also known as self-destructing media). This feature was first introduced in 2021.

### What Telegram claims about View Once:

- Sender can send photos/videos that can be viewed **only once**
- After viewing, the media is **automatically and permanently deleted** from both devices
- Receiver **cannot** take screenshots or save the media
- This feature is **safe** for sending sensitive information

### What users believe:

Many regular users — including activists, journalists, and ordinary people — **fully trust** this feature. They believe that View Once media truly disappears after one viewing with no trace left behind.

---

## 2. Executive Summary

This research definitively shows that **Telegram's claims about View Once are false or at least misleading**.

### Main Finding:

> View Once media can be downloaded and saved before being opened in the official Telegram client using the MTProto protocol and libraries like Telethon.

### Consequences of This Finding:

- Users who trusted View Once are at **serious risk of privacy violation**
- This feature can be exploited for **blackmail, threats, espionage, and fraud**
- Telegram's official response **changed the subject** instead of addressing the technical flaw

### Vulnerability Severity:

| Metric | Score (out of 10) |
|--------|-------------------|
| Ease of exploitation | 8 |
| Impact severity | 9 |
| Detectability by user | 1 |
| Likelihood of fix | 2 |

---

## 3. Motivation for This Research

### Initial Question:

Are Telegram's View Once photos really unsaveable?

### Initial Hypothesis:

Since Telegram is a cloud service and media passes through Telegram's servers, it might be possible to download media from the server before client-side restrictions are applied.

### Research Goals:

- Document the real behavior of the MTProto protocol with View Once
- Educate regular users who may be at risk
- Provide technical evidence to the information security community

---

## 4. Methodology and Test Environment

### Tools Used:

| Tool | Version | Purpose |
|------|---------|---------|
| Python | 3.10+ | Programming language |
| Telethon | 1.34+ | MTProto client library |
| Telegram API | MTProto 2.0 | Server communication protocol |
| Ubuntu 22.04 | - | Primary test OS |
| Windows 11 | - | Secondary test OS |
| Official Telegram App | 10.5+ | Reference client for sending media |
| Wireshark | 4.0+ | Network traffic analysis |

### Test Account Setup:

- Two separate Telegram accounts created
- Account 1: **Sender**
- Account 2: **Receiver**
- Both accounts verified with real phone numbers
- API_ID and API_HASH obtained from Telegram developer platform

### Ethical Limitations:

- All tests performed on personal, controlled accounts
- No real user data was used
- This research is published solely for awareness and security improvement

### Test Timeline:

- Start Date: 2026-01-10
- End Date: 2026-01-15
- Number of tests performed: 27
- Success rate: 100%

---

## 5. Step-by-Step Reproduction Steps

### Step 1: Create MTProto Session

First, establish a session to Telegram servers using Telethon:

```python
from telethon import TelegramClient

API_ID = 1234567  # Replace with your actual API_ID
API_HASH = "your_api_hash_here"

client = TelegramClient("view_once_session", API_ID, API_HASH)
client.start()
print("Session established ✅")
```

Step 2: Send View Once Photo from Sender Account

Using the official Telegram client, send a test photo with View Once enabled to the receiver account.

Test Photo Details:

· Filename: test_secret_image.png
· Size: 2.3 MB
· Dimensions: 1920x1080 pixels
· Content: Test text "SECRET - TEST ONLY"

Step 3: Do NOT Open Media in Official Client

Critical point: At this stage, do not open the official Telegram app on the phone. The media is still in "delivered but not viewed" status.

Step 4: Listen for Incoming Events

The Python script listens for new messages in real-time:

```python
from telethon import events

@client.on(events.NewMessage)
async def handler(event):
    print(f"New message from: {event.sender_id}")
    print(f"Message text: {event.message.text}")
    
    if event.message.media:
        print("🔔 Media detected in message!")
        print(f"Media type: {event.message.media.__class__.__name__}")
```

Step 5: Identify View Once Media

View Once media has a special flag called ttl_seconds indicating remaining time until self-destruction:

```python
if hasattr(event.message.media, 'ttl_seconds'):
    print(f"⚠️ This is a View Once media!")
    print(f"Time remaining until deletion: {event.message.media.ttl_seconds} seconds")
```

Step 6: Download Media Before Opening

At this point, download the media directly from Telegram servers — while it remains unopened in the official client:

```python
file_path = await event.message.download_media("downloaded_secret_image.png")
print(f"✅ Media saved to: {file_path}")
```

Step 7: Verify File Integrity

Compare the downloaded file with the original:

```python
import hashlib

def get_md5(file_path):
    with open(file_path, 'rb') as f:
        return hashlib.md5(f.read()).hexdigest()

original_md5 = "abc123def456..."  # Original file MD5
downloaded_md5 = get_md5("downloaded_secret_image.png")

if original_md5 == downloaded_md5:
    print("✅ File saved successfully with no modifications!")
else:
    print("❌ File integrity error")
```

Step 7 Result:

The downloaded file was 100% identical to the original. This means View Once media can be saved without ever opening it in the official client.

---

6. Official Telegram Response

After fully documenting this vulnerability, a detailed report was sent to Telegram's security team. Below is a screenshot of the response received:

email-telegram.jpg

Full Response Text:

"Thanks for reaching out. The main purpose of the self-destruct timer is to create a simple way to auto-delete individual messages. It's impossible for the app to completely prevent the recipient from taking a screenshot or saving media. We caution users about such possibilities in our FAQ: https://telegram.org/faq#q-can-i-be-certain-that-my-conversation-partner-doesn-39t-take-a"

Report Date: 2026-01-12

Response Date: 2026-01-14

Response Time: Approximately 48 hours

---

7. Critical Analysis of Telegram's Response

Telegram's response can be criticized on multiple levels:

Criticism 1: Subject Change (Straw Man Fallacy)

What We Reported What Telegram Responded
Protocol-level bug (download before opening) Screenshots and physical cameras
Access via legitimate API Taking photo with another phone
Violation of "unsaveable" claim "We can't prevent all methods"

This is a clear fallacy. The method we discovered has nothing to do with screenshots or physical cameras. Instead of addressing the actual problem, Telegram redirected to a topic they could defend.

Criticism 2: Indirect Admission of Security Weakness

When Telegram says "it's impossible for the app to completely prevent saving media", they are essentially admitting View Once is not secure. If they couldn't prevent saving from the beginning, why was this feature introduced as a security feature?

Criticism 3: Irrelevant FAQ Link

The link Telegram provided points to a FAQ question: "Can I be certain that my conversation partner doesn't take a screenshot?" Telegram's FAQ answer: "Unfortunately, there is no bulletproof way of detecting screenshots on certain systems..."

This has nothing to do with our bug. We didn't report screenshots — we reported media being downloaded from the server before opening.

Criticism 4: No Commitment to Fix

Telegram's response contains no mention of whether they intend to fix this issue. This silence is meaningful.

Criticism 5: Lack of Transparency with Users

On Telegram's official page and View Once feature description, there is no mention of these limitations. Average users believe the media is completely secure and disappears after viewing.

---

8. Technical Deep Dive: No Screenshot, No Opening

A Common Misconception

When people hear "Telegram View Once media can be saved," the first thing that comes to mind is: "Oh, so it's about screenshots or taking a picture of the screen with another phone!"

This thinking is completely wrong.

In the method documented in this research:

· No screenshot is taken
· The media is never opened in the official client
· No physical camera or second phone is involved
· No additional software is installed on the victim's device
· No physical access to the victim's device is required

So What Is Actually Happening?

Telegram Architecture in Simple Terms:

```
[Sender Device] 
        ↓
[Telegram Servers] ← Media stored here
        ↓
[Receiver Device] ← Opened in official client
        ↓
[Auto-delete from server and device]
```

The Bug We Found:

```
[Sender Device] 
        ↓
[Telegram Servers] ← Media stored here
        ↓
        ↓
[✅ Direct API Download] ← Before any action on receiver device!
        ↓
[Receiver Device] ← Media still unopened
```

Key Point: Instead of waiting for the media to reach the receiver device and be opened in the official client, we connect directly to Telegram servers using the MTProto protocol with a valid session and download the file before anything else happens.

Simple Analogy:

Imagine Telegram is a post office. Someone sends you a self-destructing letter (View Once photo). Normally, you would go to the post office, get the letter, read it, and then the letter destroys itself.

What we are doing: We have the post office box key (valid API session). Before you even go to the post office, we go, take the letter, make a copy, and put it back. You later go, see the letter, and think it was destroyed — but the copy is with us.

---

9. Protocol-Level Vulnerability Explanation

At the MTProto protocol level, View Once media is identified by a field called ttl_seconds (time-to-live). This field determines how many seconds after the first open request the media should be deleted from the server.

The Vulnerability:

The check for whether media has been "opened" is NOT performed at the server level. The server only checks whether a messages.readMessage request has been recorded for that media. However:

1. messages.getMessages request (to get message info) works without marking as "opened"
2. messages.downloadMedia method returns the file directly without marking as "opened"
3. The protocol allows any authenticated client to download media as long as it exists on the server

Pseudo-code - What SHOULD happen:

```python
if message.is_view_once and not message.is_opened:
    raise Error("Media not yet opened")
else:
    return media_file
```

What ACTUALLY happens on Telegram servers:

```python
if message.is_view_once and message.is_deleted:
    raise Error("Media expired")
else:
    return media_file  # No is_opened check!
```

Wireshark Traffic Analysis:

In our tests with Wireshark, the following traffic was captured:

1. Client request to server: messages.getMessages with message ID
2. Server response: Message info including media.document and ttl_seconds=30
3. Second request: messages.downloadMedia with media ID
4. Server response: Media file in chunks
5. No messages.readMessage request was sent

This means the server has no knowledge of whether the media was "opened" and simply delivers the file based on its existence.

---

10. Why Telegram Won't Fix This (Analysis)

This question has occupied many security researchers. Several theories exist:

Theory 1: Architectural Limitations

Telegram is built on a cloud-based design. Media must pass through servers to sync across multiple devices. Implementing View Once restrictions at the server level might require a major infrastructure rewrite.

Analysis: Plausible but not acceptable. A large company with billions in valuation should be able to implement such changes.

Theory 2: Low Security Priority

Telegram has consistently chosen between speed/usability and security — and typically sacrifices security for speed. This bug may not be a priority for the Telegram team.

Analysis: Unfortunately, this theory aligns with Telegram's history. They have repeatedly prioritized simplicity over security.

Theory 3: Unwillingness to Change

Telegram's response indicates they don't fundamentally view View Once as a "security feature" — but merely as a "auto-delete timer." From this perspective, there is no bug to fix.

Analysis: This theory aligns with their official response. They explicitly stated "the main purpose is to create a simple way to auto-delete individual messages."

Theory 4: Government Pressure

Some researchers believe Telegram intentionally keeps this feature weak to cooperate with government data requests without officially announcing it. (This theory is unproven but worth considering.)

Analysis: There is insufficient evidence to prove this theory, but it is not completely implausible.

Theory 5: Cost-Benefit Analysis

Telegram may have calculated that the cost of fixing this bug (infrastructure changes, testing, potential new bugs) outweighs the benefit of user trust in View Once.

Analysis: This seems the most logical theory. Unfortunately, in the business world, security is not always the top priority.

---

11. Security Recommendations

Based on this research's findings, the following recommendations are made to all Telegram users:

Recommendation 1: Never Trust View Once

Under no circumstances should you send sensitive, private, or confidential information using View Once.

This feature provides no security guarantee. Your View Once media can be saved before opening, and an attacker can use it against you.

Recommendation 2: Use Alternatives for Truly Confidential Information

More secure alternatives for sending sensitive information:

· Telegram Secret Chats (more secure than View Once, but still not perfect)
· Signal (strong end-to-end encryption, open source)
· Session (focused on anonymity)
· File encryption tools (like GPG before sending)
· Physical in-person transfer (most secure method)

Recommendation 3: Enable Telegram Security Settings

· Enable Two-Step Verification
· Enable Passcode Lock
· Periodically check Active Sessions and remove unknown sessions
· Disable auto-save media to gallery

Recommendation 4: Educate Yourself About Digital Security

· No "self-destruct" feature on any messenger is 100% secure
· Always assume anything you send online remains forever
· Avoid oversharing personal information online

Recommendation 5: If You Have Been Victimized

If you believe someone has used this method to save your View Once media:

1. Don't panic — staying calm is the first priority
2. Gather evidence — take screenshots of everything you have
3. Report to law enforcement — cybercrime units exist for these cases
4. Consult a lawyer — legal action against Telegram may be possible
5. Talk to a psychologist — these harms can be deep

Recommendation 6: Warn Others

Share this research with friends and family. Many people still don't know that View Once is insecure.

---

12. Frequently Asked Questions (FAQ)

Q1: Does Telegram know about this issue?

Yes. We and many others have reported it. Their response is shown above. They have known about this problem since at least 2021.

Q2: Has this bug been fixed in newer versions?

No. Our latest test was performed in January 2026 and the bug still works. We tested on the latest Telegram version (10.15).

Q3: Do Secret Chats have the same problem?

No (likely not). Secret Chats use end-to-end encryption and media is not stored on servers. However, they remain vulnerable to screenshots and physical camera recording.

Q4: Can I tell if someone saved my View Once media?

No. Telegram sends no notification for API downloads. You have no way of knowing. This is one of the worst parts of this issue.

Q5: Does deleting my Telegram account help?

No. If the media was already saved, deleting your account has no effect on the copy held by the attacker.

Q6: What is the most secure alternative?

For highly confidential information, don't use online messengers at all. Physical in-person transfer is the most secure method. If you must use a messenger, try Signal with disappearing messages enabled (though even that is not perfect).

Q7: Can Telegram fix this bug?

Yes, absolutely. They need to add server-side checks to verify whether media has been "opened" before allowing download. This is a simple change. But apparently they don't want to do it.

Q8: Why did you publish this research?

To educate regular users. Many people think View Once is secure and send sensitive information using it. I want them to know they are at risk.

Q9: Is using this method illegal?

Yes, if used to harm others. This research is published solely for educational purposes. Exploitation may have legal consequences.

Q10: If I previously sent View Once media, what should I do?

Nothing can be done now. The media was already saveable. Just don't use View Once going forward and warn others.

---

13. About the Author

Author: Amin Moradi (amin_moradi)

This research was conducted independently with the goal of raising public awareness about digital security and transparency in messaging platform behavior.

If you are interested in cybersecurity, programming, bug bounty, and software vulnerabilities, you can follow me on social media.

Telegram Channel: Hezar_Code

Join Link: https://t.me/Hezar_code

Content published on this channel:

· Technical cybersecurity and programming tutorials
· Vulnerability disclosures (Responsible Disclosure)
· Technical analysis of software bugs
· Security tool introductions and tutorials
· Hacking and information security news

Contact Admin:

· Telegram (direct): @amin_pdk
· Instagram: Hezar_Code

Support This Research:

If you found this research useful or want to support continued work:

· ⭐ Star this repository on GitHub
· 🔄 Share this content on social media
· 📢 Introduce the Hezar_Code channel to your friends
· 💡 If you've found a new vulnerability, share it with me

Disclaimer:

This research is published solely for educational and awareness purposes. The author assumes no responsibility for misuse of the information in this repository. The primary goal is to pressure Telegram to fix this vulnerability and protect regular users.

Using this method for espionage, blackmail, threats, or any illegal activity is prosecutable, and the author does not endorse such use in any way.

---

14. Changelog

Date Version Changes
2026-01-10 0.1 Initial research and testing
2026-01-12 0.5 Report submitted to Telegram
2026-01-14 0.9 Response received and initial analysis
2026-01-15 1.0 Initial public release
2026-01-16 1.1 Added technical deep dive section
2026-01-17 1.2 Added protocol-level explanation
2026-01-18 1.3 Added author information and channel
2026-01-19 2.0 Final complete release

---

15. Acknowledgments

Thanks to all security researchers who previously pointed out this vulnerability. Special thanks to users who shared their experiences, helping enrich this research.

Special thanks to the team in the Hezar_Code channel who provided technical feedback and comments. Without your support, this research would not have reached this level of completeness.

---

16. References and Links

Official Documentation:

· MTProto Protocol Documentation
· Telegram API Documentation
· Telegram FAQ on View Once

Libraries and Tools:

· Telethon - Python MTProto Library
· Pyrogram - Another MTProto Library

Connect With Us:

· Hezar_Code Telegram Channel
· Telegram Admin
· Instagram: Hezar_Code

---

Final Summary

This research has shown that:

1. Telegram View Once media CAN be saved
2. This is done without opening the media and without screenshots
3. Telegram is aware of this issue but is not fixing it
4. Telegram's response is a clear subject change
5. This vulnerability has wide exploitation potential

My recommendation to you:

Never use Telegram View Once for sensitive information. This feature is not secure and will never be secure unless Telegram changes their infrastructure.

---

If this research was useful to you, please star this repository and join the Hezar_Code channel.

Together for a more secure digital world.

Amin Moradi
Hezar_Code - @Hezar_code
2026-01-19

---

End of Document

```