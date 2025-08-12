<div style="font-family: Arial, sans-serif; max-width: 800px; margin: 0 auto; padding: 20px; line-height: 1.6; color: #333;">

# Developer Brief: MVP for a Kusama based Decentralised Mailing List based on Feather 

**Date Prepared:** August 12, 2025  
**Version:** 1.0  
**Authors:** Decent Partners
**Project Name:** Feather Newsletter MVP  
**Objective:** Build a minimal viable product (MVP) for a decentralised mailing list that aggregates user-submitted content into a monthly newsletter called "Feather." Submissions are made via the <a href="https://github.com/Decent-Partners/Feather-Protocol" style="color: #1a73e8; text-decoration: none;">Feather Protocol</a> on Kusama, curated by a membership-based DAO on the either the Kreivo parachain or Asset Hub (dependent on Kreivo pallet upgrade), compiled into an IPFS-hosted document, and distributed on-chain for pull-based access. The system emphasises decentralisation, permissionless submissions, and on-chain verifiability, with no reliance on centralised servers or email infrastructure.

## 1. Project Overview
<ul style="list-style-type: disc; margin-left: 20px;">
  <li><strong>Core Concept:</strong> Users submit content (text or IPFS-linked media) as "feathers" on Kusama. A Kreivo DAO curates submissions via on-chain voting. Approved content is compiled into a monthly newsletter ("Feather: Issue X"), uploaded to IPFS, and announced on-chain. Subscribers query the system to "pull" the latest issue.</li>
  <li><strong>Key Features:</strong>
    <ul style="list-style-type: circle; margin-left: 20px;">
      <li>Permissionless submissions with anti-spam via KSM burns.</li>
      <li>Decentralised curation by DAO members (membership gated but open via proposals).</li>
      <li>Pull-based distribution (like RSS) for full decentralisation.</li>
      <li>First issue: "Feather: Issue 1 – August 2025" (bootstrap with test data).</li>
    </ul>
  </li>
  <li><strong>Decentralisation Principles:</strong> All data and actions on Kusama/Kreivo. No central curator—anyone can run scripts or fork the DAO.</li>
  <li><strong>Assumptions:</strong>
    <ul style="list-style-type: circle; margin-left: 20px;">
      <li>Users have Kusama wallets (e.g., <a href="https://polkadot.js.org/apps" style="color: #1a73e8; text-decoration: none;">polkadot.js</a>).</li>
      <li>Basic IPFS access (e.g., via <a href="https://ipfs.io" style="color: #1a73e8; text-decoration: none;">ipfs.io</a> or local node).</li>
      <li>MVP focuses on text/IPFS; no advanced media handling.</li>
    </ul>
  </li>
  <li><strong>Scope Limitations (MVP):</strong>
    <ul style="list-style-type: circle; margin-left: 20px;">
      <li>No push notifications (e.g., no email/XMTP integration).</li>
      <li>Simple majority voting; no advanced curation (e.g., weighted by reputation).</li>
      <li>Handle up to 100-500 submissions/month.</li>
      <li>Testing on Kusama mainnet; use Westend for dev.</li>
    </ul>
  </li>
</ul>

## 2. Technical Components
### 2.1 Feather Protocol
<ul style="list-style-type: disc; margin-left: 20px;">
  <li><strong>Description:</strong> On-chain remark system on Kusama for publishing immutable "feathers" (short messages or links).</li>
  <li><strong>Repo:</strong> <a href="https://github.com/Decent-Partners/Feather-Protocol" style="color: #1a73e8; text-decoration: none;">https://github.com/Decent-Partners/Feather-Protocol</a></li>
  <li><strong>Usage:</strong> Submit via <code>system.remarkWithEvent</code> extrinsic (polkadot.js apps).</li>
  <li><strong>Formats for This Project:</strong>
    <ul style="list-style-type: circle; margin-left: 20px;">
      <li><strong>Submission:</strong> <code>FEATHER::monthly-contrib::Feather::[Title]::[Content or IPFS_hash]</code>
        <ul style="list-style-type: none; margin-left: 10px;">
          <li>Example: <code>FEATHER::monthly-contrib::Feather::Onchain Insights::QmX9abc123</code> (IPFS for article/image).</li>
        </ul>
      </li>
      <li><strong>Publication:</strong> <code>FEATHER::newsletter-issue::Feather::[Month Year]::[IPFS_hash]</code>
        <ul style="list-style-type: none; margin-left: 10px;">
          <li>Example: <code>FEATHER::newsletter-issue::Feather::August 2025::QmZ8ghi789</code></li>
        </ul>
      </li>
    </ul>
  </li>
  <li><strong>Cost:</strong> Minimal KSM via Dark Hole pallet (anti-spam).</li>
</ul>

### 2.2 Feather Index
<ul style="list-style-type: disc; margin-left: 20px;">
  <li><strong>Description:</strong> Local indexer for querying feathers efficiently. Builds a RocksDB database from Kusama blocks.</li>
  <li><strong>Install/Run:</strong>
    <pre style="background: #f4f4f4; padding: 10px; border-radius: 5px;">
      <code>
cargo build --release
./target/release/feather-index -u wss://kusama-rpc.polkadot.io -p 8172
      </code>
    </pre>
  </li>
  <li><strong>Query API:</strong> WebSocket at <code>ws://localhost:8172</code>.
    <ul style="list-style-type: circle; margin-left: 20px;">
      <li>Example Query (JSON): <code>{"type": "GetFeathers", "block_number": 0, "limit": 100, "genre": "monthly-contrib"}</code></li>
      <li>Filters: By genre, account_id, etc.</li>
    </ul>
  </li>
  <li><strong>Role:</strong> Aggregate submissions and issues without full-chain scans.</li>
</ul>

### 2.3 Kreivo DAO
<ul style="list-style-type: disc; margin-left: 20px;">
  <li><strong>Description:</strong> Membership-based DAO on Kreivo parachain for curation. Uses communities and referenda pallets.</li>
  <li><strong>Repo/Setup:</strong> <a href="https://github.com/virto-network/kreivo" style="color: #1a73e8; text-decoration: none;">https://github.com/virto-network/kreivo</a> (check for 2025 updates).</li>
  <li><strong>DAO Creation:</strong>
    <ul style="list-style-type: circle; margin-left: 20px;">
      <li>Register via Kreivo runtime call (~1 KSM).</li>
      <li>Name: "Feather Newsletter DAO".</li>
      <li>Membership: Stake 0.1 KSM or propose/vote for entry. Use membership pallet for roles.</li>
      <li>Governance: OpenGov-style referenda (tracks for voting).</li>
    </ul>
  </li>
  <li><strong>Voting:</strong> Aye/Nay on proposals; threshold >50%. Conviction voting for weight.</li>
  <li><strong>Integration:</strong> Scripts submit proposals and query outcomes via Substrate API.</li>
</ul>

### 2.4 Additional Tech Stack
<ul style="list-style-type: disc; margin-left: 20px;">
  <li><strong>Languages/Tools:</strong>
    <ul style="list-style-type: circle; margin-left: 20px;">
      <li>Rust: For extending feather-index if needed (e.g., add timestamps).</li>
      <li>Python: For aggregation/compilation scripts (use <code>substrate-interface</code>, <code>websocket-client</code>, <code>requests</code>, <code>ipfshttpclient</code>).</li>
      <li>Substrate API: For RPC calls (timestamps, extrinsics).</li>
      <li>IPFS: For hosting compiled newsletters (public gateways or local).</li>
    </ul>
  </li>
  <li><strong>Dependencies (Python):</strong>
    <pre style="background: #f4f4f4; padding: 10px; border-radius: 5px;">
      <code>
pip install substrate-interface websocket-client requests ipfshttpclient
      </code>
    </pre>
  </li>
  <li><strong>Dev Environment:</strong> Kusama RPC (<code>wss://kusama-rpc.polkadot.io</code>), polkadot.js for testing.</li>
</ul>

## 3. Workflow and Implementation Steps
### 3.1 Submissions
<ul style="list-style-type: disc; margin-left: 20px;">
  <li>Users submit via polkadot.js: Sign remark with format above.</li>
  <li>No code needed; educate via docs/dApp (future).</li>
</ul>

### 3.2 Aggregation and Proposal Creation
<ul style="list-style-type: disc; margin-left: 20px;">
  <li><strong>Script:</strong> <code>aggregate_and_propose.py</code> (run monthly, e.g., cron).</li>
  <li><strong>Steps:</strong>
    <ol style="margin-left: 20px;">
      <li>Query feather-index for "monthly-contrib" from last ~432,000 blocks (1 month approx).</li>
      <li>Filter precisely by timestamps (Substrate RPC).</li>
      <li>For each: Submit Kreivo referenda proposal ("Include [Title]?").</li>
    </ol>
  </li>
  <li><strong>Code Snippet (Python):</strong>
    <pre style="background: #f4f4f4; padding: 10px; border-radius: 5px;">
      <code>
import json
import websocket
from substrateinterface import SubstrateInterface, Keypair
from datetime import datetime, timedelta

kusama = SubstrateInterface(url="wss://kusama-rpc.polkadot.io")
kreivo = SubstrateInterface(url="wss://kreivo-rpc.example.com")  # Update RPC
keypair = Keypair.create_from_uri('//YourSeed')

# Query index
ws = websocket.create_connection("ws://localhost:8172")
query = {"type": "GetFeathers", "block_number": 0, "limit": 1000, "genre": "monthly-contrib"}
ws.send(json.dumps(query))
response = json.loads(ws.recv())['data']
ws.close()

# Filter last month
contributions = []
current_block = kusama.get_chain_head()['number']
month_ago_block = current_block - 432000
for feather in response:
    if feather['block_number'] < month_ago_block: continue
    # Get timestamp...
    # (Add filtering logic as in previous design)

    # Submit proposal
    proposal = f"Vote to include: {feather['remark']}"
    call = kreivo.compose_call('Referenda', 'submit', {'proposal': {'Inline': proposal.encode().hex()}, 'track': 0})
    extrinsic = kreivo.create_signed_extrinsic(call, keypair)
    kreivo.submit_extrinsic(extrinsic)
      </code>
    </pre>
  </li>
</ul>

### 3.3 Curation (Voting)
<ul style="list-style-type: disc; margin-left: 20px;">
  <li>Members vote via polkadot.js or custom UI.</li>
  <li>Voting window: 7-14 days (configure in DAO).</li>
</ul>

### 3.4 Compilation and Publication
<ul style="list-style-type: disc; margin-left: 20px;">
  <li><strong>Script:</strong> <code>compile_and_publish.py</code> (run after voting).</li>
  <li><strong>Steps:</strong>
    <ol style="margin-left: 20px;">
      <li>Query Kreivo for referenda outcomes (approved only).</li>
      <li>Compile approved into Markdown (text + IPFS embeds).</li>
      <li>Upload to IPFS, get hash.</li>
      <li>Submit publication remark on Kusama.</li>
    </ol>
  </li>
  <li><strong>Code Snippet (Extension):</strong>
    <pre style="background: #f4f4f4; padding: 10px; border-radius: 5px;">
      <code>
# After fetching approved...
newsletter_md = "# Feather: Issue 1 – August 2025\n\n"
for contrib in approved:
    # Parse and add (as before)
import ipfshttpclient
client = ipfshttpclient.connect()
ipfs_hash = client.add_str(newsletter_md)['Hash']

# Publish remark
call = kusama.compose_call('System', 'remark_with_event', {'remark': f"FEATHER::newsletter-issue::Feather::August 2025::{ipfs_hash}".encode()})
extrinsic = kusama.create_signed_extrinsic(call, keypair)
kusama.submit_extrinsic(extrinsic)
      </code>
    </pre>
  </li>
</ul>

### 3.5 Subscription/Viewer
<ul style="list-style-type: disc; margin-left: 20px;">
  <li><strong>Script:</strong> <code>view_newsletter.py</code>.</li>
  <li><strong>Steps:</strong> Query index for "newsletter-issue", fetch latest IPFS, display/save.</li>
  <li><strong>Code Snippet:</strong>
    <pre style="background: #f4f4f4; padding: 10px; border-radius: 5px;">
      <code>
# Similar query, then requests.get(f"https://ipfs.io/ipfs/{hash}")
      </code>
    </pre>
  </li>
</ul>

## 4. Development Roadmap
<ul style="list-style-type: disc; margin-left: 20px;">
  <li><strong>Phase 1 (1 Week):</strong> Set up feather-index, test submissions/queries.</li>
  <li><strong>Phase 2 (1 Week):</strong> Deploy Kreivo DAO, integrate proposal submission.</li>
  <li><strong>Phase 3 (1 Week):</strong> Build/test scripts, bootstrap first issue.</li>
  <li><strong>Phase 4 (Ongoing):</strong> Add dApp UI (React + polkadot.js), monitor costs.</li>
  <li><strong>Testing:</strong> Use Westend for end-to-end; simulate 10-20 submissions.</li>
  <li><strong>Deployment:</strong> Scripts run locally/VPS; DAO on Kreivo mainnet.</li>
</ul>

## 5. Potential Challenges and Mitigations
<ul style="list-style-type: disc; margin-left: 20px;">
  <li><strong>Chain Sync Delays:</strong> Use <code>--best</code> flag in index for faster loading.</li>
  <li><strong>Costs:</strong> Monitor KSM fees; optimise batches.</li>
  <li><strong>Spam:</strong> DAO gating + burns.</li>
  <li><strong>Scalability:</strong> Limit queries; extend index for efficiency.</li>
  <li><strong>Updates:</strong> Monitor Kusama/Kreivo upgrades (e.g., runtime changes).</li>
  <li><strong>Security:</strong> Use secure key management; audit extrinsics.</li>
</ul>

<p style="font-style: italic; color: #666;">This brief provides a blueprint for implementation. For questions, reference repos or <a href="https://docs.substrate.io" style="color: #1a73e8; text-decoration: none;">Substrate docs</a>. Estimated Build Time: 3-4 weeks for a solo dev.</p>

</div>
