## 📡 Improving PassWall2 URL Test for Iran (Instagram-Filtered Networks)

PassWall2 URL Test Optimization | OpenWrt | LuCI | Iran VPN Testing

✅ Summary

In Iran and other restricted networks, PassWall2’s default URL test (google.com/generate_204) is not enough to verify whether a VPN node can actually bypass filters for apps like Instagram.

This guide explains:
	•	How the URL test works internally
	•	Where the test URL is hardcoded
	•	Why Instagram.com itself is a bad test target
	•	How to replace it with a better endpoint
	•	How to automate the change safely

The result is a more realistic VPN node test that reflects real-world Instagram connectivity in Iran.

⸻

🔍 Problem

PassWall2 uses this URL by default:

https://www.google.com/generate_204

This endpoint:
	•	Only checks basic connectivity
	•	Does not confirm filter bypass
	•	Can show false positives for VPN nodes in Iran

At the same time:
	•	instagram.com ❌ blocks HEAD requests
	•	Has heavy anti-bot protections
	•	Produces unreliable curl results

⸻

🎯 Goal

Test whether a VPN node can actually access Instagram-class filtered services
➡ without false negatives
➡ without modifying PassWall2 core logic

⸻

🧠 Investigation Process (How We Found It)

1️⃣ Found the LuCI button

In LuCI, the URL Test button is defined here:

/usr/lib/lua/luci/view/passwall2/global/status.htm

The button triggers:

/cgi-bin/luci/admin/services/passwall2/urltest_node


⸻

2️⃣ Traced LuCI → Lua → Shell

That endpoint maps to a Lua controller which executes:

/usr/share/passwall2/test.sh url_test_node


⸻

3️⃣ Found the actual test URL

Inside:

/usr/share/passwall2/test.sh

The function url_test_node() uses:

curl -I -skL https://www.google.com/generate_204

This is the real URL behind the “URL Test” button.

⸻

✅ Solution: Use Meta CDN Instead of Instagram.com

❌ Why NOT instagram.com
	•	Blocks HEAD
	•	Returns 403 / 429
	•	Anti-bot & rate-limited
	•	Unstable for automated tests

✅ Why edge-mqtt.facebook.com is ideal
	•	Used by Instagram / Meta services
	•	Filtered in Iran
	•	Lightweight
	•	Accepts HEAD
	•	Stable for curl
	•	Accurately reflects Instagram reachability

🎯 Final replacement

https://edge-mqtt.facebook.com


⸻

🛠️ Manual Changes

1️⃣ Replace default URL test (node list button)

File

/usr/share/passwall2/test.sh

Replace

https://www.google.com/generate_204

With

https://edge-mqtt.facebook.com


⸻

2️⃣ Replace “desired ping address” in PassWall2 basic settings

File

/usr/lib/lua/luci/view/passwall2/global/status.htm

Edit the top of the page and replace the ping / test address with:

https://edge-mqtt.facebook.com

(This affects the basic settings page, not just node testing.)

⸻

⚙️ Automation Script (AI-Generated)

⚠️ Note:
The following script is AI-generated.
Please review it before running in production.

passwall2-instagram-urltest-fix.sh

#!/bin/sh

echo "[+] Updating PassWall2 URL test for Iran / Instagram filtering"

TEST_SH="/usr/share/passwall2/test.sh"
STATUS_HTML="/usr/lib/lua/luci/view/passwall2/global/status.htm"

OLD_URL="https://www.google.com/generate_204"
NEW_URL="https://edge-mqtt.facebook.com"

# Backup
cp "$TEST_SH" "${TEST_SH}.bak"
cp "$STATUS_HTML" "${STATUS_HTML}.bak"

# Replace URL in test.sh
sed -i "s|$OLD_URL|$NEW_URL|g" "$TEST_SH"

# Replace ping/test address in LuCI status page
sed -i "s|$OLD_URL|$NEW_URL|g" "$STATUS_HTML"

chmod +x "$TEST_SH"

echo "[✓] URL test updated successfully"
echo "[i] Backups created:"
echo "    - ${TEST_SH}.bak"
echo "    - ${STATUS_HTML}.bak"
echo "[i] Please refresh LuCI or restart uhttpd if needed"

Run it:

sh passwall2-instagram-urltest-fix.sh


⸻

🚀 Result

After applying this:
	•	URL Test button reflects real Instagram access
	•	VPN nodes that can’t bypass filters fail correctly
	•	No false positives
	•	No Instagram anti-bot issues

⸻

🔑 SEO Keywords (for discoverability)
	•	PassWall2 URL test
	•	OpenWrt VPN Iran
	•	Instagram filtered Iran VPN
	•	LuCI PassWall2 optimization
	•	OpenWrt VPN node testing
	•	Bypass Iran internet filtering
	•	PassWall2 generate_204 replacement
	•	Instagram connectivity test OpenWrt

⸻

🤝 Need More?

I can:
	•	Find more hidden endpoints
	•	Add multi-URL fallback testing
	•	Make the URL configurable via UCI
	•	Add region-specific tests
	•	Patch other LuCI / PassWall internals

📩 Just contact me — I’m happy to dig deeper.
