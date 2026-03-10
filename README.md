import requests
import re
import sys

if len(sys.argv) != 3:
    print("Usage: python github_subdomains.py <domain> <github_token>")
    sys.exit()

domain = sys.argv[1]
token = sys.argv[2]

headers = {
    "Authorization": f"token {token}",
    "Accept": "application/vnd.github.v3.text-match+json"
}

query = f'"{domain}"'
url = f"https://api.github.com/search/code?q={query}&per_page=100"

subdomains = set()

print(f"[+] Searching GitHub for {domain}...\n")

while url:
    r = requests.get(url, headers=headers)
    data = r.json()

    if "items" not in data:
        break

    for item in data["items"]:
        file_url = item["html_url"]
        raw_url = item["html_url"].replace("github.com", "raw.githubusercontent.com").replace("/blob/", "/")

        try:
            content = requests.get(raw_url).text
            matches = re.findall(rf"[a-zA-Z0-9.-]+\.{re.escape(domain)}", content)

            for sub in matches:
                subdomains.add(sub)

        except:
            pass

    if "next" in r.links:
        url = r.links["next"]["url"]
    else:
        break

print("\n[+] Subdomains Found:\n")

for sub in sorted(subdomains):
    print(sub)

print(f"\nTotal: {len(subdomains)}")# SubDomain-Enumeraion
