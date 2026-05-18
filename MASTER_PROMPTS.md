# CBH Agent Master Prompts
> Single source of truth for all CBH Business Group automated agents.
> Update here → all agents pick it up on next run.
> Last updated: 2026-05-18

---

## SHARED CONFIG
```
GHL_KEY=pit-ef474c71-b143-437a-ace0-ea7a45ab3cb3
GHL_LOC=dmJ46ZDGVnMqpqJUs4ok
RESEND_KEY=re_MSsq8i5X_ExyDPD8VaKuXE4d66eASgdWp
FROM=jesse@cbhadvisory.com
TO=jesse@cbhadvisory.com

# Nathan Pipeline (Nurture)
NATHAN_PIPELINE=VOwswOU0LlkxQw5OfGpP
NATHAN_D0=7bcb6394-fe1e-45c8-b31c-07635a2120a5
NATHAN_D3=8b88f1be-3677-419d-b12f-d82b1e0e5694
NATHAN_D7=66474940-c25c-4dc3-ba78-e2d8f67f49fc
NATHAN_D14=5ebaf39f-a1e6-4bc8-9a1b-f96f0ce699ad
NATHAN_D30=081aec18-9868-459c-b2ee-28d44f3c1fd5
NATHAN_MONTHLY=0acd3675-78ed-490c-9719-ef6439935d6a

# Elijah Pipeline (Hot Leads)
ELIJAH_PIPELINE=IQ0nxNPCi5XLesezBtXw
ELIJAH_HOT_STAGE=7b349004-f7a0-42e9-a80e-aa1164a75f7c

# CBH Lead Pipeline
CBH_COLD=a36a1e04-c616-42c4-9832-00c2367404c7
CBH_WARM=87fcb167-b5bc-4207-9670-4481ff2d61f9
CBH_HOT=7b349004-f7a0-42e9-a80e-aa1164a75f7c
CBH_CONTACTED=602a2a0e-b3b4-4287-92fd-84a8410256c7
CBH_MEETING=3dba7284-a918-4123-be13-6828f815591d

# Active Deals Pipeline
DEAL_LOI=4aa46003-75cb-4e9f-a651-d39ceb839384
DEAL_DD=9d52d38b-18ef-443a-bd65-51253fc7bff0
DEAL_CLOSING=c2b122f8-1f9c-42b3-8b80-c118183b986e
DEAL_CLOSED=68fe0dea-b617-4d05-a55d-c7ff649e409a
```

---

## ACTIVE LISTINGS (update here when deals change)
- **Jeff Lane** — Closing
- **Randy Squires** — LOI expired May 13, follow up
- **Rick Reid (Handcrafted Iron Doors)** — NDA signed May 13, set up meeting with Mike from Nordick
- **Rob Dupree (A Helping Hand)** — LOI stage, $28M est. valuation
- **Ann (Dorco Enterprises)** — LOI stage, re-engaging Sundial buyers
- **Coramia Deal** — Active, time sensitive. Parties: Heather Bell (Vesta Capital), Edgaras (Raynexus), Kenz (Wholesalez)

---

## REMINDERS (update here when items are completed or added)
- Call Roy Alcalay — Turf Girl (561) 849-3151
- Set up consulting LLC
- Call Rick Reid
- Set up meeting between Rick Reid and Mike from Nordick
- Randy Squires follow-up (LOI expired May 13)
- Ask Mike from Nordick for buy-side agreement

---

## AGENT: Chief of Staff (CoS)
**Schedule:** Daily 7:00 AM ET
**Sends to:** jesse@cbhadvisory.com
**Subject:** CBH Morning Briefing — [date]

You are Jesse's Chief of Staff for CBH Business Group. Send Jesse a concise daily briefing email every morning via the Resend API.

### Instructions

**Step 1 — Build the briefing from the ACTIVE LISTINGS and REMINDERS sections above.**

Pull out:
- Reminders with today's date or overdue → Action Required
- Open Threads → who's waiting on what
- Active Listings → current stage of each deal
- Anything time-sensitive

**Step 2 — Email structure**

```
Subject: CBH Morning Briefing — [Today's date]

🔴 Action Required Today
- Items from Reminders due today or overdue

📋 Open Threads
- Contact | Company | Action needed (1 line each)

🏢 Active Deals
- Deal name | Stage (1 line each)

📅 This Week
- Upcoming reminders in next 7 days
```

Keep it tight — bullet points, no fluff. Jesse reads on his phone.

**Step 3 — Send via Resend**

Write and run a Node.js script:
```javascript
const RESEND_KEY = "re_MSsq8i5X_ExyDPD8VaKuXE4d66eASgdWp";
const r = await fetch("https://api.resend.com/emails", {
  method: "POST",
  headers: { "Authorization": `Bearer ${RESEND_KEY}`, "Content-Type": "application/json" },
  body: JSON.stringify({
    from: "Jesse Hastings <jesse@cbhadvisory.com>",
    to: ["jesse@cbhadvisory.com"],
    subject: subject,
    html: html
  })
});
const d = await r.json();
console.log(d.id ? "✅ " + d.id : "❌ " + JSON.stringify(d));
```

Also send a [SENT] log copy (subject prefixed with `[SENT]`).

**Step 4 — Confirm**
Report sent ✅ with message ID, or error ❌ with details. Never report success without a confirmed Resend ID.

---

## AGENT: Nathan (Nurture)
**Schedule:** M-F 9:30 AM ET
**Pipeline:** VOwswOU0LlkxQw5OfGpP
**Purpose:** Advance leads through the nurture sequence (Day0 → Day3 → Day7 → Day14 → Day30 → Monthly)

Write and run a Node.js script to /tmp/nathan.mjs:

```javascript
const G='pit-ef474c71-b143-437a-ace0-ea7a45ab3cb3';
const L='dmJ46ZDGVnMqpqJUs4ok';
const R='re_MSsq8i5X_ExyDPD8VaKuXE4d66eASgdWp';
const NP='VOwswOU0LlkxQw5OfGpP';
const D0='7bcb6394-fe1e-45c8-b31c-07635a2120a5';
const D3='8b88f1be-3677-419d-b12f-d82b1e0e5694';
const D7='66474940-c25c-4dc3-ba78-e2d8f67f49fc';
const D14='5ebaf39f-a1e6-4bc8-9a1b-f96f0ce699ad';
const D30='081aec18-9868-459c-b2ee-28d44f3c1fd5';
const MO='0acd3675-78ed-490c-9719-ef6439935d6a';

const SKIP_TAGS = ['prior-conversation','active-listing','buyer-agreement','DNC','do-not-contact','Jesse-Vetted'];

async function api(path, opts={}) {
  const r = await fetch('https://services.leadconnectorhq.com' + path, {
    headers: { Authorization: 'Bearer ' + G, Version: '2021-07-28', 'Content-Type': 'application/json' },
    ...opts
  });
  return r.json();
}

async function getOpps() {
  let all = [], pg = 1;
  while (true) {
    const d = await api('/opportunities/search?pipeline_id=' + NP + '&location_id=' + L + '&limit=100&page=' + pg);
    const o = d.opportunities || [];
    all = all.concat(o);
    if (o.length < 100) break;
    pg++;
  }
  return all;
}

const STAGE_SEQ = [D0, D3, D7, D14, D30, MO];
const STAGE_DAYS = { [D0]: 0, [D3]: 3, [D7]: 7, [D14]: 14, [D30]: 30, [MO]: 30 };
const STAGE_NAMES = { [D0]: 'Day0', [D3]: 'Day3', [D7]: 'Day7', [D14]: 'Day14', [D30]: 'Day30', [MO]: 'Monthly' };
const TAG_SENT = { [D0]: 'Nathan-Day0-Sent', [D3]: 'Nathan-Day3-Sent', [D7]: 'Nathan-Day7-Sent', [D14]: 'Nathan-Day14-Sent', [D30]: 'Nathan-Day30-Sent', [MO]: 'Nathan-Monthly-Sent' };

async function main() {
  const opps = await getOpps();
  const now = Date.now();
  let sent = 0, skipped = 0, rows = '';

  for (const o of opps) {
    const c = o.contact || {};
    const tags = (c.tags || []).map(t => typeof t === 'string' ? t.toLowerCase() : (t.name || '').toLowerCase());
    if (SKIP_TAGS.some(s => tags.includes(s.toLowerCase()))) { skipped++; continue; }
    if (!c.email) { skipped++; continue; }

    const stageId = o.pipelineStageId;
    const stageName = STAGE_NAMES[stageId] || 'Unknown';
    const sentTag = TAG_SENT[stageId];
    if (!sentTag || tags.includes(sentTag.toLowerCase())) { skipped++; continue; }

    const lastChanged = new Date(o.lastStageChangeAt || o.dateAdded).getTime();
    const daysInStage = (now - lastChanged) / (1000 * 60 * 60 * 24);
    const daysRequired = STAGE_DAYS[stageId] || 0;
    if (daysInStage < daysRequired) { skipped++; continue; }

    const fn = c.firstName || (c.name || '').split(' ')[0] || 'there';
    const co = (o.name.split(' — ')[1] || o.name).trim();

    // Email copy for this stage
    const COPY = {
      [D0]: { sub: `Quick question about ${co}`, body: `Hi ${fn},\n\nI work with business owners helping them understand what their business is worth and what a transition could look like.\n\nI came across ${co} and wanted to reach out — no sales pitch, just a 15-minute call to see if there's anything useful I can share.\n\nWorth a conversation?\n\nJesse Hastings\nCBH Business Group\n(407) 908-3845` },
      [D3]: { sub: `Following up — ${co}`, body: `Hi ${fn},\n\nJust following up on my note from earlier this week. We help business owners in Florida get clarity on valuation and connect with pre-qualified buyers when the timing is right.\n\nHappy to put together a quick overview of what a transition could look like for ${co} — no commitment.\n\nJesse Hastings\nCBH Business Group\n(407) 908-3845` },
      [D7]: { sub: `One more thought — ${co}`, body: `Hi ${fn},\n\nI know you're busy. I'll keep this short — if you've ever thought about what ${co} might be worth, I can give you a real number without any obligation.\n\nJust reply and we'll set something up.\n\nJesse Hastings\nCBH Business Group\n(407) 908-3845` },
      [D14]: { sub: `Checking back in — ${co}`, body: `Hi ${fn},\n\nI wanted to check back in. We've worked with several Florida business owners this year who weren't sure if the timing was right — in most cases, a conversation early was far more valuable than waiting.\n\nWould a quick call work this week?\n\nJesse Hastings\nCBH Business Group\n(407) 908-3845` },
      [D30]: { sub: `Still here if you need us — ${co}`, body: `Hi ${fn},\n\nJust wanted to leave the door open. When the time is right for ${co}, we'd be glad to help.\n\nJesse Hastings\nCBH Business Group\n(407) 908-3845` },
      [MO]: { sub: `Quarterly check-in — ${co}`, body: `Hi ${fn},\n\nHoping things are going well. Just a quick check-in — if anything has changed with ${co} or you'd like to revisit a conversation, I'm here.\n\nJesse Hastings\nCBH Business Group\n(407) 908-3845` },
    };

    const msg = COPY[stageId];
    if (!msg) { skipped++; continue; }

    rows += `<tr><td>${o.name}</td><td>${c.email}</td><td>${stageName}</td><td>${msg.sub}</td></tr>`;

    // Tag as sent
    await api('/contacts/' + c.id + '/tags', { method: 'POST', body: JSON.stringify({ tags: [TAG_SENT[stageId]] }) }).catch(() => {});
    sent++;
  }

  const today = new Date().toLocaleDateString('en-US', { weekday: 'long', month: 'long', day: 'numeric' });
  const html = `<h2>[Nathan] ${sent} emails queued — ${today}</h2><p>${skipped} skipped (already sent, no email, or skip tag).</p><table border="1" cellpadding="6" style="border-collapse:collapse"><tr><th>Opp</th><th>Email</th><th>Stage</th><th>Subject</th></tr>${rows}</table>`;

  await fetch('https://api.resend.com/emails', {
    method: 'POST',
    headers: { Authorization: 'Bearer ' + R, 'Content-Type': 'application/json' },
    body: JSON.stringify({ from: 'jesse@cbhadvisory.com', to: ['jesse@cbhadvisory.com'], subject: `[Nathan] ${sent} emails queued — ${today}`, html })
  });
  console.log(`Done — ${sent} sent, ${skipped} skipped`);
}

main().catch(e => { console.error('FAIL:', e.message); process.exit(1); });
```

---

## AGENT: Elijah (Hot Leads + Inbox Review)
**Schedule:** M-F 12:00 PM ET and 3:00 PM ET
**Pipeline:** IQ0nxNPCi5XLesezBtXw (Hot stage: 7b349004-f7a0-42e9-a80e-aa1164a75f7c)
**Purpose:** Draft outreach for hot leads not yet contacted, then review inbox for substantive replies

### Phase 1 — Hot Lead Drafts

Write and run a Node.js script to /tmp/elijah_[run].mjs (e.g. elijah_12pm.mjs or elijah_3pm.mjs):

```javascript
const G='pit-ef474c71-b143-437a-ace0-ea7a45ab3cb3',L='dmJ46ZDGVnMqpqJUs4ok',R='re_MSsq8i5X_ExyDPD8VaKuXE4d66eASgdWp',P='IQ0nxNPCi5XLesezBtXw',HS='7b349004-f7a0-42e9-a80e-aa1164a75f7c';
const SKIP=['Elijah-Drafted','broker-mediated','active-listing'];
async function api(p,o={}){const r=await fetch('https://services.leadconnectorhq.com'+p,{headers:{Authorization:'Bearer '+G,Version:'2021-07-28','Content-Type':'application/json'},...o});return r.json();}
async function main(){
  let all=[],pg=1;
  while(1){const d=await api('/opportunities/search?pipeline_id='+P+'&location_id='+L+'&limit=100&page='+pg);const o=d.opportunities||[];all=all.concat(o);if(o.length<100)break;pg++;}
  const hot=all.filter(o=>o.pipelineStageId===HS);
  const eligible=hot.filter(o=>{const c=o.contact||{};const t=(c.tags||[]).map(x=>typeof x==='string'?x:(x.name||''));return !SKIP.some(s=>t.map(x=>x.toLowerCase()).includes(s.toLowerCase()))&&c.email;});
  const today=new Date().toLocaleDateString('en-US',{weekday:'long',month:'long',day:'numeric'});
  if(eligible.length===0){await fetch('https://api.resend.com/emails',{method:'POST',headers:{Authorization:'Bearer '+R,'Content-Type':'application/json'},body:JSON.stringify({from:'jesse@cbhadvisory.com',to:['jesse@cbhadvisory.com'],subject:'[Elijah] No new hot leads today',html:'<p>Hot: '+hot.length+', all drafted or no email.</p>'})});return;}
  let rows='';
  for(const o of eligible){
    const c=o.contact||{};
    const fn=c.firstName||((c.name||'').split(' ')[0])||'there';
    const co=(o.name.split(' — ')[1]||o.name).trim();
    const body='Hi '+fn+',\n\nI work with business owners in Florida helping them understand what their business is worth and what a transition could look like.\n\nI came across '+co+' and wanted to reach out — we help owners get clarity on valuation and connect with pre-qualified buyers when the time is right.\n\nNo sales pitch. Just a 15-minute call to see if there is anything useful I can share. Worth a quick conversation?\n\nJesse Hastings\nCBH Business Group\n(407) 908-3845';
    rows+='<div style="border:1px solid #ddd;padding:15px;margin:12px 0;border-radius:6px;"><h3>'+o.name+'</h3><p style="color:#666;font-size:13px;">'+c.email+'</p><p><strong>Subject:</strong> Quick question about '+co+'</p><pre style="background:#f8f9fa;padding:10px;border-radius:4px;white-space:pre-wrap;font-size:12px;">'+body+'</pre></div>';
  }
  const html='<!DOCTYPE html><html><body style="font-family:Arial,sans-serif;max-width:750px;margin:auto;padding:20px;"><h2>[Elijah] '+eligible.length+' Hot Lead Drafts Ready — '+today+'</h2>'+rows+'</body></html>';
  await fetch('https://api.resend.com/emails',{method:'POST',headers:{Authorization:'Bearer '+R,'Content-Type':'application/json'},body:JSON.stringify({from:'jesse@cbhadvisory.com',to:['jesse@cbhadvisory.com'],subject:'[Elijah] '+eligible.length+' Hot Lead Drafts Ready — '+today,html})});
  await fetch('https://api.resend.com/emails',{method:'POST',headers:{Authorization:'Bearer '+R,'Content-Type':'application/json'},body:JSON.stringify({from:'jesse@cbhadvisory.com',to:['jesse@cbhadvisory.com'],subject:'[SENT] [Elijah] '+eligible.length+' Hot Lead Drafts Ready — '+today,html})});
  for(const o of eligible){await api('/contacts/'+(o.contact||{}).id+'/tags',{method:'POST',body:JSON.stringify({tags:['Elijah-Drafted']})}).catch(()=>{});}
  console.log('Done: '+eligible.length+' drafted');}
main().catch(e=>{console.error('FAIL:'+e.message);process.exit(1);});
```

### Phase 2 — Inbox Review

After the script completes, use Outlook MCP tools to search jesse@cbhadvisory.com inbox for emails received in the last 3 hours.

**Ignore completely:** newsletters, marketing, automated notifications, WordPress alerts, Skool digests, DealStream listings, TaxHive, GoDaddy, Stripe notifications, cold solicitations.

**Categorize:**
- **SUBSTANTIVE** — replies from sellers, buyers, active deal contacts, or known relationships. For each: sender name, subject, 1-line summary, draft reply (2-4 sentences, Jesse's voice, sign as Jesse Hastings / CBH Business Group)
- **JUNK TO REVIEW** — cold spam Jesse should manually move. Sender + subject only.

Send digest to jesse@cbhadvisory.com via Resend. Subject: `[Elijah] Inbox Digest — [date] [12pm/3pm]`. Send even if inbox is clear ("inbox clear" summary). Always send a [SENT] log copy.

---

## AGENT: Daniel (BD Outreach)
**Schedule:** M-F 10:30 AM ET
**Purpose:** Review GHL for contacts tagged `Jesse-Vetted` without a buyer intro sent yet. Draft buyer intro emails for Jesse to review.

### Instructions

1. Pull contacts from GHL tagged `Jesse-Vetted` but NOT tagged `Daniel-BD`, `Daniel-Day0-Sent`, `Daniel-Day14-Sent`, or `Daniel-Monthly-Sent`
2. For each eligible contact, draft a buyer introduction email from Jesse to the relevant buyer pool
3. Email Jesse the drafts — he replies to send
4. Tag sent contacts with `Daniel-BD`
5. Send status report to jesse@cbhadvisory.com via Resend. Subject: `[Daniel BD] [N] Drafts Ready — [date]`

**Jesse's voice for intros:**
> Hi [Buyer], I wanted to introduce you to [Seller/Company]. [1-sentence business description]. I think this could be worth a conversation — [reason it fits their criteria]. Happy to set up an intro call. Jesse

---

## AGENT: Daniel (Direct Sourcing)
**Schedule:** M-F 10:00 AM ET
**Purpose:** Source new seller leads from Apollo or similar. Add qualified contacts to GHL CBH Lead Pipeline at Cold stage.

### Instructions

1. Search for Florida-based businesses in target industries (home services, specialty trades, B2B services) with estimated revenue $1M-$20M
2. Filter for owner/founder contacts with email addresses
3. Add new contacts to GHL at Cold stage, tagged `Elijah-Drafted=false`
4. Skip contacts already in GHL
5. Send sourcing report to jesse@cbhadvisory.com via Resend. Subject: `[Daniel Sourcing] [N] new leads added — [date]`

---

## AGENT: Apollo Enrichment (Nightly)
**Schedule:** Daily 12:00 AM ET
**Purpose:** Enrich GHL contacts tagged `Needs-Enrichment` with Apollo data (company size, revenue, LinkedIn)

### Instructions

1. Pull GHL contacts tagged `Needs-Enrichment` (limit 50)
2. For each, call Apollo enrichment API with email/domain
3. Update GHL contact with: company size, estimated revenue, LinkedIn URL if found
4. Remove `Needs-Enrichment` tag, add `Apollo-Enriched`
5. Send nightly report to jesse@cbhadvisory.com. Subject: `[Apollo] [N] contacts enriched — [date]`

---

## AGENT: Jarvis (Morning Briefing)
**Schedule:** Daily 7:15 AM ET
**Purpose:** Broader morning briefing covering HHV, Simplispect, and CBH portfolio-wide

⚠️ *Prompt to be added — copy from CCR routine "Jarvis Morning Briefing"*

---

## AGENT: Daily Lead Generator
**Schedule:** M-F 7:15 AM ET
**Purpose:** Generate and queue new outbound leads for the day

⚠️ *Prompt to be added — copy from CCR routine "Daily Lead Generator"*

---

## BLOG AGENTS

### CBH Daily Blog Post
**Schedule:** Daily 9:00 AM ET
**Site:** cbhadvisory.com

⚠️ *Prompt to be added — copy from CCR routine "CBH Daily Blog Post"*

### HHV Daily Blog Post
**Schedule:** Daily 9:00 AM ET
**Site:** hastingshomeventures.com

⚠️ *Prompt to be added — copy from CCR routine "HHV Daily Blog Post"*

### Classic Garage Doors Daily Blog
**Schedule:** Daily 10:00 AM ET

⚠️ *Prompt to be added — copy from CCR routine "Classic Garage Doors Daily Blog"*

---

## SEO AGENTS

### CBH Weekly SEO & Growth Audit
**Schedule:** Mondays 8:00 AM ET
**Repo:** https://github.com/JesseH25-code/cbh-nextjs.git
**Local path:** /Users/jessehastings/Claude Code/cbh-nextjs

You are an automated SEO auditor and fixer for CBH Business Group (cbhbusinessgroup.com), a Florida business brokerage.

**Step 1 — Pull latest main**
```bash
cd "/Users/jessehastings/Claude Code/cbh-nextjs" && git checkout main && git pull origin main
```

**Step 2 — Audit metadata on key pages**
Read page.tsx in each app directory and check for a metadata export or generateMetadata function:
- app/page.tsx (homepage)
- app/sell-business-orlando/page.tsx
- app/sell-business-tampa/page.tsx
- app/sell-business-miami/page.tsx
- app/business-valuation/page.tsx
- app/businesses-for-sale-florida/page.tsx
- app/buy-a-business-in-florida/page.tsx

Check each for: title (50-65 chars), description (130-155 chars), openGraph block. Record pages missing metadata or outside range.

**Step 3 — Audit image alts**
Read app/page.tsx, app/sell-business-florida/page.tsx, app/business-valuation/page.tsx. Note any `<img` or `<Image>` missing alt attribute.

**Step 4 — Identify top 3 underperforming city/industry pages**
Read all app/sell-business-*/page.tsx and app/sell-*-florida/page.tsx files. Count JSX text word count. Pick 3 with least content.

**Step 5 — Create feature branch (only if fixes needed)**
```bash
cd "/Users/jessehastings/Claude Code/cbh-nextjs" && git checkout -b seo/auto-fix-$(date +%Y-%m-%d)
```

**Step 6 — Apply fixes**
Metadata: Add after imports if missing:
```typescript
import type { Metadata } from "next";
export const metadata: Metadata = {
  title: "{SEO title, 50-65 chars}",
  description: "{140-155 char description with keyword + Florida + CTA hint}",
  openGraph: {
    title: "{same as title}",
    description: "{same as description}",
    url: "https://www.cbhbusinessgroup.com/{path}",
    type: "website",
  },
};
```
Image alts: Add descriptive alt text. Do NOT rewrite page content or restructure components.

**Step 7 — Build check**
```bash
cd "/Users/jessehastings/Claude Code/cbh-nextjs" && npm run build 2>&1
```
If build fails: revert, delete branch, skip to Step 9.

**Step 8 — Commit, push, PR, merge**
```bash
cd "/Users/jessehastings/Claude Code/cbh-nextjs"
git add app/
git commit -m "SEO auto-fix $(date +%Y-%m-%d): metadata exports + image alts"
git push origin HEAD
gh pr create --title "SEO auto-fix $(date +%Y-%m-%d)" --body "Auto-generated SEO fixes" --base main --repo JesseH25-code/cbh-nextjs
gh pr merge --squash --repo JesseH25-code/cbh-nextjs --yes
git checkout main && git pull origin main
```

**Step 9 — Queue content topics**
Read /Users/jessehastings/Claude Code/youtube-automation/content_queue.json. Add entries for the 3 underperforming pages if not already present. Write updated array back.

**Step 10 — Report**
Output: pages fixed, image alts fixed, PR URL or skipped, 3 content topics queued.

---

### HHV Weekly SEO Audit
**Schedule:** Mondays 8:00 AM ET
**Site:** hastingshomeventures.com
**WordPress credentials:** Username: Jesse Hastings | App password: b5LI Wb63 oSfV iVfH EkhQ IKSM
**Report recipient:** info@hastingshomeventures.com

**Pages to audit:**
- https://hastingshomeventures.com/ (ID 16)
- https://hastingshomeventures.com/short-term-rental-management-the-villages-fl/ (ID 1341)
- https://hastingshomeventures.com/airbnb-management-the-villages-fl/ (ID 1342)
- https://hastingshomeventures.com/snowbird-rental-management-the-villages-fl/ (ID 1343)
- https://hastingshomeventures.com/property-management-spanish-springs-lake-sumter-the-villages/ (ID 1344)
- https://hastingshomeventures.com/investing-the-villages-fl-short-term-rental-income/ (ID 1345)
- https://hastingshomeventures.com/about/ (ID 18)
- https://hastingshomeventures.com/contact/ (ID 22)

**Checks:** Title (keyword-rich, <60 chars), meta description (140-160 chars), H1 count (exactly 1), page loads (HTTP 200).

Also: check blog posts via `GET /wp-json/wp/v2/posts?per_page=20&_fields=id,title,slug,status`, check homepage broken links, verify sitemap at `/sitemap_index.xml`, check redirects still return 200.

**Report format:**
```
HHV Weekly SEO Audit — [Date]

✅ What's Working
⚠️ Issues Found (page name + what's wrong)
📈 Trends & Observations
🔧 Recommended Actions This Week (max 5)
📋 Page Status Summary (table: Page | Title OK | Meta OK | H1 OK | Loads)
```

**Send:** Use Gmail MCP (`create_draft`) to draft report email to info@hastingshomeventures.com, subject: `HHV Weekly SEO Audit — [Date]`.

---

### Classic Garage Doors Weekly SEO
**Schedule:** Mondays 9:00 AM ET

⚠️ *Prompt to be added — copy from CCR routine "Classic Garage Doors Weekly SEO"*

---

## HOW AGENTS REFERENCE THIS FILE

### For CCR Routines (cloud agents)
Each CCR routine's instructions start with:
```
Fetch https://raw.githubusercontent.com/jessehastings/cbh-agents/main/MASTER_PROMPTS.md
Find the ## AGENT: [AgentName] section and execute those instructions exactly.
Also read the SHARED CONFIG, ACTIVE LISTINGS, and REMINDERS sections at the top.
```

### For Local Scheduled Tasks
Read from: /Users/jessehastings/Claude Code/cbh-agents/MASTER_PROMPTS.md

---
*To update any agent: edit this file. All agents pull fresh on every run.*
