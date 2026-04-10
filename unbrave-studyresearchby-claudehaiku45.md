# Unbraved Chromium: Comprehensive Research Study
## Removing Brave-Specific Features & Designing Custom Implementations

**Research Date**: April 2026  
**Project**: unbraved-chromium-meta  
**Scope**: Complete analysis of Brave Browser feature removal, replacement strategies, and rebranding  
**Inspiration**: Ungoogled Chromium approach to feature curation and removal

---

## Executive Summary

Brave Browser is built on Chromium with significant proprietary extensions. This research documents how to systematically remove or replace Brave-specific features while maintaining a functional, privacy-respecting browser. The goal is to create **unbraved-chromium-meta**: a Chromium derivative focused on privacy, user control, and extensibility for custom service implementations.

### Key Findings

- **8+ Major Brave Features** identified for removal/replacement: Sync, Accounts, Leo AI, Wallet, Rewards, News, VPN, and branded Shields
- **Modularity Assessment**: Most user-facing features (Wallet, Leo, Rewards, News) are semi-isolated and can be disabled or replaced
- **Core Integration**: Sync/Account infrastructure and Shields are tightly coupled to Chromium rendering pipeline
- **Backend Dependency**: Brave relies on external services (sync servers, AI APIs, wallet RPC nodes) that must be disintermediated
- **Removal Strategy**: Remove Brave backends, expose configuration hooks, enable custom implementations

### Architecture Overview

```
Chromium Core
├── Brave Shields (Privacy/Blocking) [CORE, tightly integrated]
├── Brave Sync (Cross-device sync) [CORE, tightly coupled]
├── Brave Accounts (Auth/user mgmt) [CORE, infrastructure]
├── Brave Leo (AI Assistant) [MODULAR, WebUI-based]
├── Brave Wallet (Web3/Crypto) [MODULAR, WebUI-based]
├── Brave Rewards (Ad monetization) [MODULAR, settings-based]
├── Brave News (Content feed) [MODULAR, NTP component]
├── Brave VPN (Proxy service) [MODULAR, settings-based]
└── Other (Translate, Talk, Playlist) [LOW PRIORITY]
```

### Removal Complexity Matrix

| Feature | Type | Coupling | Difficulty | Priority | Replacement Strategy |
|---------|------|----------|-----------|----------|----------------------|
| **Brave Sync** | Core | Very High | HARD | 1️⃣ Critical | Stub out API, design plugin architecture |
| **Brave Accounts** | Infrastructure | Very High | HARD | 1️⃣ Critical | Remove auth enforcement, make optional |
| **Brave Leo** | User-facing | Medium | MEDIUM | 2️⃣ High | Disable UI, design AI service abstraction |
| **Brave Wallet** | User-facing | Medium | MEDIUM | 2️⃣ High | Disable UI, design wallet service interface |
| **Brave Rewards** | User-facing | Medium | MEDIUM | 3️⃣ Medium | Remove entirely, can't be easily replaced |
| **Brave News** | NTP Component | Low | EASY | 3️⃣ Medium | Remove from NTP, disable feature flag |
| **Brave VPN** | Modular | Low | EASY | 4️⃣ Lower | Disable settings panel, stub implementation |
| **Branded Shields** | Core | Very High | HARD | 2️⃣ High | Rebrand, keep core privacy tech |
| **IPFS Support** | Removed | Low | MEDIUM | 4️⃣ Lower | Document restoration path |

---

## Part 1: Codebase Structure

### Repository Layout

**Primary Repository** (brave-browser): Issue tracking, changelogs, wiki only
- Status: Meta/tracking repository
- Files: CHANGELOG_DESKTOP.md, CHANGELOG_ANDROID.md, CHANGELOG_iOS.md, README, package.json
- Source code: Located in brave-core (sister repository)

**Source Repository** (brave-core): Actual browser implementation
- Location: https://github.com/brave/brave-core
- Language: C++, JavaScript/TypeScript, Python
- Build System: GN (Chromium's build system)
- Platform Support: Windows, macOS, Linux, Android, iOS

### Key Directory Structure (brave-core)

```
brave-core/
├── browser/
│   ├── brave_shields/      [Ad blocking, tracker blocking, fingerprinting]
│   ├── sync/               [Brave Sync implementation]
│   ├── accounts/           [User authentication & device management]
│   ├── web3/               [Brave Wallet, Web3 integration]
│   ├── ai/                 [Brave Leo AI assistant]
│   ├── rewards/            [Brave Rewards monetization]
│   ├── news/               [Brave News feed]
│   └── vpn/                [Brave VPN integration]
├── components/
│   ├── brave_extensions/   [Feature flag implementations]
│   ├── brave_wallet/       [Wallet WebUI & logic]
│   ├── leo/                [Leo AI WebUI & logic]
│   └── [other modules]
├── third_party/
│   └── [external libraries, RPC endpoints, API stubs]
├── chromium/              [Chromium submodule]
└── BUILD.gn              [Build configuration]
```

### Build Configuration

**GN Build System**: Chromium's build system with Brave-specific flags
```
# Key Brave feature flags in build configuration
- enable_brave_shields=true     [Privacy & blocking layer]
- enable_brave_sync=true        [Cross-device sync]
- enable_brave_leo=true         [AI assistant]
- enable_brave_wallet=true      [Web3 wallet]
- enable_brave_rewards=true     [Ad monetization]
- enable_brave_news=true        [Content feed]
- enable_brave_vpn=true         [VPN integration]
```

These flags allow conditional compilation of features at build time.

---

## Part 2: Deep Dive into Brave-Specific Features

### 1. Brave Sync (Cross-Device Synchronization)

#### Purpose & Status
Brave Sync enables users to sync bookmarks, history, passwords, and settings across multiple devices without creating an account or sending data to Brave servers.

**Current Status** (v1.89+): 
- Fully integrated into browser core
- Uses peer-to-peer protocol with code-based device pairing
- Enabled by default for new instances
- Password syncing is opt-in but recommended

#### Architecture & Implementation

**Components**:
- `brave/browser/sync/` — Sync engine and device pairing logic
- `brave/components/brave_sync/` — Sync protocol implementation
- Native UI at `brave://settings/sync` — settings panel for sync management
- Device pairing code generation and validation
- Bookmark, history, password, and settings serialization

**Brave-Specific Integration Points**:
1. **Sync Service URL**: Currently hardcoded to Brave's sync servers
   - Configuration point: `--sync-url=` flag at runtime
   - Default: `https://sync.bravesoftware.com`
   
2. **Device Identity & Authentication**:
   - Peer-to-peer code generation (8-word device sync codes)
   - Device certificate generation and validation
   
3. **Data Sync Chain**:
   - Encrypted sync chain stored locally
   - Sync metadata stored in profile directory (`~/.config/BraveSoftware/Brave-Browser/Default/Sync Data/`)
   
4. **Settings Integration**:
   - Sync preferences stored in `brave://settings` preferences
   - Feature flag: `brave_sync::prefs::kSyncEnabled`

#### Removal Strategy (HARD - Core Feature)

**Why It's Difficult**:
- Tightly coupled to browser startup and shutdown
- Embedded in user profile initialization
- Authentication used for other Brave services

**Recommended Approach**:
1. **Phase 1: Configuration Disabling**
   - Remove `brave://settings/sync` UI elements
   - Add environment variable/flag to completely disable sync service: `--disable-brave-sync`
   - Remove sync checks from startup sequence (stub out API endpoints)

2. **Phase 2: API Abstraction**
   - Create `SyncService` interface/abstract class
   - Implement stub/no-op version of sync service
   - Design plugin system for alternative sync implementations
   - Document how to inject custom sync providers

3. **Phase 3: Custom Implementation Hook**
   - Design extension point for third-party sync services
   - Example: Allow configuration of custom sync backend via extension/plugin
   - Support for standard protocols: WebDAV, Nextcloud, custom REST APIs

**Code Removal Checklist**:
- ✓ Disable Sync settings UI (`brave/ui/views/brave_sync_settings.cc`)
- ✓ Remove sync initialization from profile loading (`brave/browser/profile_resetter.cc`)
- ✓ Stub out sync service API calls (replace with no-op implementations)
- ✓ Remove device pairing code UI (`brave/ui/webui/sync/`)
- ✓ Clean up sync data migration code for old profiles

**Files to Modify/Remove**:
- `brave/browser/sync/` — Entire sync engine (REMOVE or STUB)
- `brave/browser/ui/webui/sync/` — Sync UI webpages (REMOVE)
- `brave/components/brave_sync/` — Sync protocol (REMOVE or REPLACE)
- Settings references: Search `kSyncEnabled`, `sync_url` in codebase

#### Extensibility Design

```cpp
// Proposed architecture for custom sync providers
class SyncProvider {
  virtual bool IsEnabled() = 0;
  virtual void SyncBookmarks(const BookmarkList&) = 0;
  virtual void SyncHistory(const HistoryList&) = 0;
  virtual void SyncPasswords(const PasswordList&) = 0;
  virtual BookmarkList RetrieveBookmarks() = 0;
  // ... other sync operations
};

// Configuration: Allow plugin system or environment variable
// for custom sync service injection
```

---

### 2. Brave Accounts (Authentication & Device Management)

#### Purpose & Status
Brave Accounts manages user identity, device recognition, and ties various Brave services together (originally planned for unified auth, now primarily for device management and Rewards).

**Current Status** (v1.89+):
- Used for Rewards linking
- Device fingerprinting
- User profile identification
- Ad engagement tracking (via Rewards system)

#### Architecture & Implementation

**Components**:
- `brave/browser/accounts/` — Account management core
- `brave/components/brave_accounts/` — Authentication logic
- Settings integration at `brave://settings/system` (for Privacy/Ads)
- Device identity generation and storage

**Brave-Specific Integration Points**:
1. **Account Authentication**:
   - OAuth-like flow to Brave servers
   - Cached credentials in profile directory
   - Session tokens for API requests

2. **Device Identity**:
   - Unique device fingerprint/ID generation
   - Stored in local preferences/profile
   - Used for analytics and Rewards verification

3. **Service Coupling**:
   - Rewards system depends on authenticated device identity
   - P3A (privacy-preserving analytics) sends anonymized data with device ID
   - Sync may depend on account context (in some configurations)

4. **Settings & Preferences**:
   - Account status in browser settings
   - Link/unlink buttons for services
   - Privacy settings tied to account identity

#### Removal Strategy (HARD - Infrastructure)

**Why It's Difficult**:
- Other features depend on account infrastructure
- Device identity used throughout the codebase
- Authentication required for some onboarding flows

**Recommended Approach**:
1. **Phase 1: Make Account Optional**
   - Create stub/no-op account service
   - Generate local device ID (don't tie to Brave servers)
   - Remove Brave account linking UI from settings
   - Disable onboarding screens that require account creation

2. **Phase 2: Decouple Features**
   - Audit each feature that depends on accounts
   - Replace account ID with locally-generated unique ID
   - Remove remote account verification

3. **Phase 3: Abstract Authentication**
   - Design pluggable authentication interface
   - Allow users to plug in custom account backends (self-hosted, Mozilla, etc.)
   - Or completely disable authentication if not needed

**Code Removal Checklist**:
- ✓ Remove `brave://accounts` UI pages
- ✓ Remove account linking from settings panels
- ✓ Disable account API calls to Brave servers
- ✓ Generate local device ID instead of fetching from server
- ✓ Remove onboarding flows requiring accounts
- ✓ Audit and mock out all account dependencies

**Files to Modify**:
- `brave/browser/accounts/` — Account logic (REMOVE or REPLACE)
- `brave/components/brave_accounts/` — Auth implementation (STUB)
- `brave/browser/ui/views/` — Settings UI for accounts (REMOVE panels)
- Dependency audit: Search `BraveAccountService`, `GetBraveAccount()`, `AccountState`

#### Extensibility Design

```cpp
// Proposed interface for pluggable authentication
class AuthenticationProvider {
  virtual bool IsAuthenticated() = 0;
  virtual std::string GetDeviceID() = 0;
  virtual std::string GetAccessToken() = 0;
  virtual void SignIn(const std::string& user) = 0;
  virtual void SignOut() = 0;
};

// Allow environment variable or config file to specify custom provider
// Default: LocalAuthProvider (no remote authentication)
```

---

### 3. Brave Leo (AI Assistant)

#### Purpose & Status
Brave Leo is an AI-powered assistant integrated into the browser sidebar, providing conversations, summarizations, and code assistance powered by multiple LLM providers.

**Current Status** (v1.89+):
- WebUI-based (JavaScript/React frontend)
- Supports multiple AI models:
  - Claude Opus/Sonnet (Anthropic)
  - DeepSeek (local/remote)
  - Qwen (Alibaba)
  - Gemma (Google)
  - Llama (Meta)
- "Bring Your Own Model" (BYOM) support for Ollama
- Integration with Brave Search for search-grounded responses
- Tab/page context awareness with text selection
- Image upload, PDF attachment support
- Conversation history management
- Accessible via `chrome://leo-ai` and side panel UI

#### Architecture & Implementation

**Components**:
- `brave/components/leo/` — Leo frontend (WebUI, TypeScript/React)
- `brave/browser/ai/` — Leo backend integration, API communication
- `brave/browser/ui/webui/` — WebUI hosting for Leo interface
- Settings: `brave://settings/leo-ai` configuration panel
- AI API endpoints: Proxied through Brave infrastructure

**Brave-Specific Integration Points**:
1. **API Endpoints**:
   - Conversations sent to Brave API gateway
   - Brave gateway proxies to selected LLM provider
   - Configuration: Model selection, API key management (for BYOM)

2. **Model Registry**:
   - Default models configured server-side
   - Model list fetched from Brave servers
   - Premium model availability linked to account

3. **Search Integration**:
   - Brave Search API integration for search-grounded responses
   - Search query attribution in responses

4. **Branding**:
   - "Brave Leo" naming throughout UI
   - Brave logo and theming
   - Integration with Brave shields/privacy indicators

5. **Data Handling**:
   - Conversation history stored locally or in sync
   - User preferences stored in profile
   - Optional privacy-preserving telemetry

#### Removal Strategy (MEDIUM - Modular)

**Why It's Manageable**:
- WebUI-based implementation is relatively isolated
- API communication is clear and mockable
- Can be disabled via feature flag
- No deep rendering pipeline integration

**Recommended Approach**:
1. **Phase 1: Feature Flag Disable**
   - Add global feature flag: `--disable-leo` or `--disable-brave-ai`
   - Remove Leo UI from sidebar, settings
   - Feature flag: `brave_leo::features::kLeoEnabled`

2. **Phase 2: Design AI Service Abstraction**
   - Create `AIServiceProvider` interface
   - Move current Leo implementation to `BraveAIServiceProvider`
   - Allow plugging in alternative AI services
   - Support for local LLM backends (Ollama, LLaMA.cpp, etc.)

3. **Phase 3: Implement Stub**
   - Stub Leo WebUI to show "AI features disabled" message
   - Design configuration system for custom AI providers
   - Or completely remove the component

**Code Removal Checklist**:
- ✓ Remove `chrome://leo-ai` WebUI page
- ✓ Remove Leo sidebar icon and menu items
- ✓ Remove `brave://settings/leo-ai` settings panel
- ✓ Disable API calls to Brave Leo endpoint
- ✓ Remove model registry fetching
- ✓ Remove Brave Search integration from Leo
- ✓ Disable conversation history sync

**Files to Modify/Remove**:
- `brave/components/leo/` — Leo frontend (REMOVE)
- `brave/browser/ai/` — Leo backend (REMOVE or REPLACE)
- `brave/browser/ui/webui/leo/` — Leo WebUI (REMOVE)
- Build config: Remove `enable_brave_leo` flag
- Search for: `kLeoEnabled`, `BraveLeoService`, `leo_api_`

#### Extensibility Design

```cpp
// Proposed architecture for pluggable AI services
class AIServiceProvider {
  virtual bool IsAvailable() = 0;
  virtual std::string GetProviderId() = 0;
  virtual std::string SendMessage(const std::string& message,
                                   const std::vector<std::string>& context) = 0;
  virtual bool SupportsSearchGrounding() = 0;
  virtual std::string SearchAndRespond(const std::string& query) = 0;
};

// Configuration: Allow users to specify AI provider
// Examples:
// --ai-provider=ollama --ollama-endpoint=http://localhost:11434
// --ai-provider=openai --openai-key=sk-xxx
// --ai-provider=disabled (default in unbraved)
```

**Custom Implementation Example**: Ollama Integration
- Users can run Ollama locally on their machine
- Configure unbraved-chromium to connect to local Ollama instance
- Get LLM capabilities without sending data to Brave or any cloud service
- Full privacy preservation

---

### 4. Brave Wallet (Web3/Crypto Management)

#### Purpose & Status
Brave Wallet is a built-in cryptocurrency wallet supporting multiple blockchains and providing DApp interaction capabilities.

**Current Status** (v1.89+):
- **Blockchains Supported**:
  - EVM chains (Ethereum, Polygon, Optimism, Arbitrum, Avalanche, Celo, Gnosis, Cronos, Harmony, Fantom, Aurora, Zksync, Linea, Manta, Base, Blast, Mode, Op Sepolia, Sepolia Testnet)
  - Solana
  - Bitcoin (Taproot support)
  - Zcash (Shield/unshield transactions)
  - Cardano (Plutus scripts, native token transactions)
  - Filecoin
- **Wallet Features**:
  - Self-custody (seed phrase generation and management)
  - Hardware wallet integration (Ledger, Trezor)
  - Custodial account options (Stripe, PayPal, Gemini, ZebPay)
  - Token swaps (LiFi, Jupiter, 0x providers)
  - Bridge transactions
  - NFT portfolio management
  - Transaction simulation
  - Dapp connection and signing

#### Architecture & Implementation

**Components**:
- `brave/components/brave_wallet/` — Wallet WebUI (TypeScript/React)
- `brave/browser/brave_wallet/` — Wallet backend, blockchain interaction
- Wallet settings: `brave://settings/wallet`
- Wallet UI: Side panel, pop-up interactions with DApps

**Brave-Specific Integration Points**:
1. **Blockchain RPC Endpoints**:
   - Proxied through Brave infrastructure by default
   - Chainstack RPC proxies for EVM chains
   - Brave's own proxies for Zcash and other chains
   - Configuration point: Can specify custom RPC endpoints per chain

2. **Wallet Branding**:
   - "Brave Wallet" naming and logo
   - Brave-specific onboarding flows
   - Brave-specific recovery mechanisms

3. **Account Linking**:
   - Custodial wallet providers (Stripe, PayPal) integrated with Brave Accounts

4. **Privacy Features**:
   - Wallet activities don't require account login
   - Private key encryption with local password
   - Optional privacy-preserving analytics

#### Removal Strategy (MEDIUM - Modular)

**Why It's Manageable**:
- Wallet is a semi-isolated WebUI component
- RPC endpoints are configurable
- Can be disabled via feature flag
- Core blockchain logic can be extracted

**Recommended Approach**:
1. **Phase 1: Feature Flag Disable (Easiest)**
   - Build flag: `enable_brave_wallet=false`
   - Remove wallet UI from sidebar, new tab page
   - Feature flag: `brave_wallet::features::kBraveWalletEnabled`

2. **Phase 2: Replace RPC Infrastructure**
   - Replace Brave RPC proxies with public alternatives:
     - Infura, Alchemy, or QuickNode for EVM chains
     - Or locally-hosted RPC nodes
   - Update blockchain interaction layer to use configurable endpoints
   - Allow users to specify custom RPC providers

3. **Phase 3: Rebrand or Replace**
   - Option A: Rebrand to "unbraved-wallet" with custom branding
   - Option B: Remove entirely and recommend third-party wallets
   - Option C: Keep wallet logic, route through custom service provider

**Code Removal Checklist** (if fully removing):
- ✓ Remove `brave/components/brave_wallet/` from build
- ✓ Remove wallet sidebar icon and menu items
- ✓ Remove wallet settings panel
- ✓ Remove DApp connection support from content scripts
- ✓ Remove crypto currency conversion features

**Code Modification Checklist** (if replacing RPC infrastructure):
- ✓ Identify all RPC endpoint configurations (`brave_wallet::rpc::`)
- ✓ Create custom RPC endpoint configuration system
- ✓ Update blockchain query methods to use configurable endpoints
- ✓ Rebrand UI elements if keeping wallet

**Files to Modify**:
- `brave/components/brave_wallet/` — Wallet frontend (REMOVE or REBRAND)
- `brave/browser/brave_wallet/` — Wallet backend (MODIFY RPC endpoints)
- Build config: Remove `enable_brave_wallet` flag
- Search for: `BraveWalletService`, `RPCEndpoint`, `rpc_url`

#### Extensibility Design

```cpp
// Proposed architecture for custom wallet RPC providers
class WalletRPCProvider {
  virtual std::string GetEndpointForChain(const std::string& chain_id) = 0;
  virtual std::string SendRPC(const std::string& method,
                              const std::string& params) = 0;
  virtual bool SupportsChain(const std::string& chain_id) = 0;
};

// Configuration example:
// --wallet-rpc-provider=custom
// --wallet-rpc-config=/path/to/rpc-config.json
// 
// rpc-config.json format:
// {
//   "ethereum": "http://localhost:8545",
//   "solana": "http://localhost:8899",
//   "bitcoin": "http://my-bitcoin-node:18332"
// }
```

---

### 5. Brave Rewards (Ad Monetization System)

#### Purpose & Status
Brave Rewards is an ad monetization system where users are rewarded with BAT (Basic Attention Token) for viewing privacy-preserving ads and can tip their favorite creators.

**Current Status** (v1.89+):
- Ad serving and tracking (privacy-preserving)
- BAT token rewards
- Creator monetization
- Payout schedules
- Multiple custodial providers (Stripe, PayPal, Gemini, ZebPay)
- Self-custody via Solana
- Redemption options

#### Architecture & Implementation

**Components**:
- `brave/browser/rewards/` — Rewards engine and reconciliation logic
- Settings: `brave://rewards` panels and notifications
- Native UI for ad dashboard
- Rewards notifications in browser UI

**Brave-Specific Integration Points**:
1. **Ad Catalog & Source**:
   - Ads fetched from Brave Ad Catalog
   - Ad impressions recorded locally
   - Privacy-preserving analytics

2. **Token Distribution**:
   - BAT token issuance tied to Brave Rewards program
   - Payout processing through Brave custodians
   - Self-custody wallet integration

3. **Creator Support**:
   - Creator registration tied to Brave identity
   - Tipping infrastructure dependent on Brave Rewards

4. **Settings & Preferences**:
   - Ad frequency settings
   - Payout schedule configuration
   - Tied to Brave Accounts for identity

#### Removal Strategy (HARD - Remove Completely)

**Why It's Difficult to Keep**:
- Entire system built around BAT token and Brave ecosystem
- Cannot be easily "replaced" with custom implementation
- Requires active participation in Brave infrastructure
- Regulatory/compliance considerations (financial rewards)

**Recommended Approach**:
1. **Complete Feature Removal** (Recommended)
   - Remove Rewards engine entirely
   - Remove rewards settings panels
   - Remove ad serving infrastructure
   - Build flag: `enable_brave_rewards=false`

2. **Why Not Replace**:
   - Would require separate blockchain/token system
   - Regulatory complexity (paying users)
   - Creator monetization requires centralized infrastructure
   - Too Brave-specific to maintain independently

**Code Removal Checklist**:
- ✓ Remove `brave/browser/rewards/` directory entirely
- ✓ Remove `brave://rewards` WebUI pages
- ✓ Remove ad serving and impression tracking
- ✓ Remove payout and token reconciliation logic
- ✓ Remove creator support infrastructure
- ✓ Remove notifications related to rewards/ads
- ✓ Disable all Rewards settings panels

**Files to Remove**:
- `brave/browser/rewards/` — All rewards logic
- `brave/browser/ui/webui/rewards/` — Rewards UI
- `brave/components/brave_rewards/` — Rewards components
- Build config: Remove `enable_brave_rewards` flag
- Search for: `RewardsService`, `BraveRewards`, `ads::service`

#### Why Not Custom Replacement

Unlike Sync or Leo, there's no practical way to replace Brave Rewards with a custom implementation:
1. **Token Economics**: Would need your own blockchain/token
2. **Regulatory/Compliance**: Paying users for content requires licensing
3. **Creator Infrastructure**: Requires centralized creator registry and payout system
4. **Ad Network**: Would need to build your own ad network and ad quality controls
5. **Financial Custody**: Requires regulated financial infrastructure

**Recommendation**: Accept this as a feature completely removed and removed. If users want rewards/monetization, they can use different browsers or complementary tools.

---

### 6. Brave News (Content Feed)

#### Purpose & Status
Brave News is a content feed shown on the New Tab Page, with news sources curated by region/language preferences.

**Current Status** (v1.89+):
- Regional news sources
- Customization via settings
- Privacy-preserving (no tracking)
- Brave API for content discovery

#### Architecture & Implementation

**Components**:
- `brave/browser/ntp_background_images/` and related — News feed logic
- WebUI component on New Tab Page
- Settings for news customization

**Brave-Specific Integration Points**:
1. **Content Source**:
   - Articles fetched from Brave News API
   - Branding: "Brave News" in UI

2. **Customization**:
   - Stored in profile preferences
   - Brave's content categories and sources

#### Removal Strategy (EASY - Modular)

**Why It's Easy**:
- Self-contained NTP component
- Can be disabled with feature flag
- No deep dependencies

**Recommended Approach**:
1. **Feature Flag Disable**:
   - Build flag: `enable_brave_news=false`
   - Feature flag: `brave_news::features::kBraveNewsEnabled`
   - OR simply remove from New Tab Page rendering

2. **Complete Removal**:
   - Remove news section from NTP UI
   - Remove news settings
   - Remove news API communication

**Code Removal Checklist**:
- ✓ Remove news section from New Tab Page
- ✓ Remove news settings panel
- ✓ Disable news API calls
- ✓ Remove news component from build

**Files to Modify**:
- `brave/browser/ntp_background_images/` — Disable news logic
- New Tab Page WebUI — Remove news rendering
- Build config: Remove news-related flags
- Search for: `kBraveNewsEnabled`, `news_api`, `BraveNews`

---

### 7. Brave VPN

#### Purpose & Status
Brave VPN is a subscription VPN service integrated into browser settings.

**Current Status** (v1.89+):
- Server location selection
- Subscription management
- Tied to Brave Accounts
- Performance optimizations

#### Architecture & Implementation

**Components**:
- Settings panel at `brave://settings/security`
- `brave/browser/vpn/` — VPN connection logic
- Native integration with system VPN APIs

**Brave-Specific Integration Points**:
1. **VPN Server Selection**:
   - Server list from Brave VPN infrastructure
   - Location-based routing

2. **Account Integration**:
   - Subscription tied to Brave Account
   - Authentication via Brave backend

#### Removal Strategy (EASY - Remove)

**Why It's Easy**:
- Self-contained feature
- Can be disabled with flag
- No critical browser functionality depends on it

**Recommended Approach**:
1. **Complete Removal**:
   - Remove VPN settings panel
   - Remove VPN infrastructure
   - Build flag: `enable_brave_vpn=false`

2. **Alternative**: Disable and recommend third-party VPN tools

**Code Removal Checklist**:
- ✓ Remove `brave/browser/vpn/` directory
- ✓ Remove VPN settings UI from `brave://settings`
- ✓ Disable VPN API calls
- ✓ Remove VPN subscription code

**Files to Remove**:
- `brave/browser/vpn/` — VPN logic
- Build config: Remove `enable_brave_vpn`
- Search for: `VPNService`, `BraveVPN`

---

### 8. Branded Shields & Privacy Features (Rebrand, Not Remove)

#### Purpose & Status
Brave Shields is the core privacy and content blocking system, including ad blocking, tracker blocking, fingerprinting protection, HTTPS upgrade, and script blocking.

**Current Status** (v1.89+)**:
- **Procedural cosmetic filtering** (advanced way to hide ads)
- **Tracker blocking** with parameter removal
- **HTTPS upgrade** (force HTTPS where possible)
- **Fingerprinting protection** (spoof fingerprinting vectors)
- **Script blocking** (3rd party scripts)
- **Cookie blocking** (1st party/3rd party separation)
- **Per-site customization** (granular control)
- **Global Privacy Control** (GPC)

#### Architecture & Implementation

**Components**:
- `brave/browser/brave_shields/` — Core Shields implementation
- `brave/components/` — Shields UI and logic
- Settings: `brave://settings/shields`
- Per-site settings: Shields UI in address bar

**Brave-Specific Elements**:
- Naming: "Brave Shields"
- Branding: Shields icon and UI
- Default configurations optimized by Brave

#### Rebrand Strategy (REBRAND, NOT REMOVE)

**Why Keep This**:
- Core to privacy-respecting browser
- One of Brave's best features
- Chromium doesn't include this by default
- Worth keeping and rebranding

**Recommended Approach**:
1. **Rebrand UI Elements**:
   - Change "Brave Shields" → "unbraved Shields" (or project-specific name)
   - Update shield icon design to match new branding
   - Change from Brave logo to new project logo
   - Update UI strings (run through translation system again)

2. **Keep Core Technology**:
   - Keep all privacy/blocking functionality
   - Keep procedural cosmetic filtering
   - Keep tracker blocking and parameter removal
   - Keep fingerprinting protection

3. **Update Configuration**:
   - Adjust default block lists (can customize)
   - Optionally include different ad/tracker lists
   - Configurable via extension system

**Code Modification Checklist**:
- ✓ Change "Brave Shields" string to new name in all UI
- ✓ Replace shield icons with new branding
- ✓ Update settings pages (`brave://settings/shields` → customize URL)
- ✓ Update colors/theming to match new brand
- ✓ Update documentation strings mentioning "Brave"

**Files to Modify**:
- `brave/browser/ui/webui/` — Settings UI (update strings)
- `brave/components/brave_shields_ui/` — Shields UI (rebrand)
- `brave/resources/` — Icons and images (replace Brave logo)
- Search for: "Brave Shields", shield icon references, Brave branding in UI

---

## Part 3: Backend Services & Infrastructure

### Brave Service Architecture

Brave relies on several backend services that must either be removed, replaced, or stubbed:

#### 1. **Sync Service** (`sync.bravesoftware.com`)
- **Purpose**: Synchronization coordination between devices
- **Endpoints**:
  - Device pairing
  - Sync chain management
  - Data resolution and conflict handling
- **Removal Strategy**: Stub or replace with custom sync backend

#### 2. **Account Service** (`api.bravesoftware.com`)
- **Purpose**: User authentication, device registration, service linking
- **Endpoints**:
  - Authentication tokens
  - Device registration
  - Account status
- **Removal Strategy**: Remove authentication requirement or replace with local auth

#### 3. **Leo AI Service** (`api.bravesoftware.com/ai`)
- **Purpose**: AI conversation processing
- **Endpoints**:
  - Conversation submission
  - Model selection
  - Response generation (proxied to LLM providers)
- **Removal Strategy**: Replace with local or alternative AI provider

#### 4. **Wallet RPC Proxies**
- **Purpose**: Blockchain interaction
- **Endpoints**:
  - Chainstack RPC proxies for EVM
  - Brave RPC for Zcash and other chains
- **Removal Strategy**: Replace with public RPC endpoints or local nodes

#### 5. **News API** (`news.bravesoftware.com`)
- **Purpose**: Content discovery for News feed
- **Endpoints**:
  - Article listing
  - Source categorization
- **Removal Strategy**: Remove or replace with alternative news source

#### 6. **Rewards Service** (`ledger-verification.herokuapp.com`, `api.uphold.com`)
- **Purpose**: Token distribution, payout processing
- **Endpoints**:
  - Reconciliation
  - Payout processing
- **Removal Strategy**: Remove entirely

#### 7. **VPN Infrastructure**
- **Purpose**: VPN server management and connection
- **Endpoints**:
  - Server list
  - VPN authentication
- **Removal Strategy**: Remove entirely

### Configuration & API Endpoint Customization

**How Brave Configures Endpoints**:
- Compile-time constants in `BUILDFLAG()` macros
- Runtime configuration via `prefs::` (preferences system)
- Command-line flags for testing/override
- Environment variables for CI/CD

**Customization Points**:

```cpp
// Example: Configuring custom sync endpoint
// In brave/browser/sync/brave_sync_service.cc:

// Default (built-in):
const char kDefaultSyncURL[] = "https://sync.bravesoftware.com";

// Making it customizable:
std::string GetSyncURL() {
  // Check environment variable first
  const char* env_url = getenv("BRAVE_SYNC_URL");
  if (env_url) return env_url;
  
  // Check user preferences
  if (pref_service_->HasPrefPath("sync.custom_url"))
    return pref_service_->GetString("sync.custom_url");
  
  // Check command line
  if (base::CommandLine::ForCurrentProcess()->HasSwitch("sync-url"))
    return base::CommandLine::ForCurrentProcess()->GetSwitchValueASCII("sync-url");
  
  // Default
  return kDefaultSyncURL;
}
```

**Priority for Configuration** (highest to lowest):
1. Command-line flags (`--custom-service-url=`)
2. Environment variables (`BRAVE_CUSTOM_SERVICE_URL=`)
3. User preferences stored in profile
4. Compiled defaults

---

## Part 4: Rebranding Checklist

### Strings & UI Text

**Search & Replace Candidates**:

| Term | Replace With | Scope |
|------|--------------|-------|
| "Brave Browser" | "unbraved Chromium Meta" | All UI, settings, help text |
| "Brave Shields" | "unbraved Shields" or "Privacy Shields" | Settings, UI panels |
| "Brave Wallet" | "unbraved Wallet" or "Chromium Wallet" | Wallet UI, settings |
| "Brave Leo" | "unbraved AI" or "Chromium Assistant" | AI UI, settings |
| "Brave News" | "Content Feed" or remove | NTP, settings |
| "Brave Sync" | "unbraved Sync" or "Device Sync" | Sync settings, UI |
| "Brave Rewards" | [Remove entirely] | — |
| "Brave VPN" | [Remove entirely] | — |
| Brave logo | New project logo | Icons, assets, UI |
| brave:// URLs | unbraved:// or chrome:// | All internal URLs |
| "Brave Software" | "unbraved Project" | Legal, help text |

**Tools for Bulk Replacement**:
```bash
# Search for all "Brave" occurrences in source
grep -r "Brave" brave-core/brave/browser --include="*.cc" --include="*.h" --include="*.ts" --include="*.js"

# Replace in batch (use with care!)
find brave-core/brave -type f \( -name "*.cc" -o -name "*.h" -o -name "*.ts" \) \
  -exec sed -i 's/Brave Browser/unbraved Chromium Meta/g' {} +
```

### Icons & Assets

**Locations to Replace**:
- `brave/browser/resources/` — Traditional assets
- `brave/ui/images/` — UI icons
- `brave/app/resources/` — App branding images
- `android/java/res/` — Android images
- `ios/brave/resources/` — iOS branding

**Approach**:
1. Create new icon designs matching new branding
2. Replace all `.png`, `.svg`, `.webp` files
3. Update resource references (may need `.grd` file updates)
4. Test on all platforms (Windows, macOS, Linux, Android, iOS)

### URLs & API References

**Update API Endpoint References**:
- `https://brave.com` → New project website
- `https://sync.bravesoftware.com` → Custom sync or removed
- `https://api.bravesoftware.com` → Custom API or removed
- `https://news.bravesoftware.com` → Custom or removed
- `https://ledger-verification.herokuapp.com` → Remove
- `community.brave.app` → New community site (or remove)

**Search References**:
```bash
grep -r "brave\\.com\\|bravesoftware\\.com\\|brave:\\/\\/" brave-core/brave \
  --include="*.cc" --include="*.h" --include="*.ts" --include="*.json"
```

### Build Configuration & Feature Flags

**Update GN Build Flags**:
```gn
# In build configuration file
declare_args() {
  # Existing Brave flags to modify or remove
  enable_brave_shields = true        # Keep but rebrand
  enable_brave_sync = false          # Disable/replace
  enable_brave_wallet = false        # Customize or remove
  enable_brave_leo = false           # Remove
  enable_brave_rewards = false       # Remove
  enable_brave_news = false          # Remove
  enable_brave_vpn = false           # Remove
  
  # New feature flags for unbraved customization
  enable_custom_sync_provider = true
  enable_ai_service_abstraction = true
  enable_wallet_service_abstraction = true
}
```

### Translations & Localization

**Transifex Project**:
- Currently: `https://explore.transifex.com/brave/brave_en/`
- For unbraved: Create new Transifex project or use local strings
- All UI strings in `.grd` files and translations
- Update source language first, then rollout to translators

---

## Part 5: IPFS Support Restoration

### Historical Context

IPFS (InterPlanetary File System) support was included in early versions of Brave but was removed in v1.69.153 due to:
- Low usage
- Maintenance burden
- Complexity in core browser
- Updates to IPFS node requirements

### Current Status

- **IPFS Local Node Support**: Removed
- **IPFS Gateway Support**: Partially available via address bar
- **WebRTC Support**: Still in place (needed for IPFS DHT)

### Restoration Strategy

#### Option 1: IPFS Gateway (Easiest)

Use public IPFS gateways instead of local node:

```
ipfs://CID → https://gateway.ipfs.io/ipfs/CID
ipns://name.eth → https://gateway.ipfs.io/ipns/name.eth
```

**Implementation**:
1. Add IPFS protocol handler in address bar
2. Redirect to public gateway (configurable)
3. No local node required
4. Minimal performance impact

**Code Changes Needed**:
- `brave/browser/protocol_handler/` — Add IPFS handler
- Rewrite `ipfs://` and `ipns://` URLs to gateway URLs
- Configuration for custom gateway endpoint

#### Option 2: Local IPFS Node (Complex)

Integrate a local IPFS node like original Brave implementation:

**Dependencies**:
- js-ipfs or go-ipfs binary
- Integration with browser process
- Lifecycle management (start on browser launch, cleanup on exit)

**Implementation Complexity**:
- High: Requires spawning external process
- Medium: Binary distribution and updates
- Medium: Integration with browser shutdown

**Code Changes Needed**:
- `brave/browser/ipfs/` — New IPFS service
- Process spawning and lifecycle management
- Integration with address bar and tab loading
- DHT and network peer discovery

#### Option 3: Hybrid Approach

- Local node for advanced users
- Gateway fallback for others
- Configuration in `brave://settings/ipfs`

**Recommended**: Option 1 (Gateway) for simplicity, with Option 2 (Local Node) as stretch goal

### IPFS Implementation Checklist

**Phase 1: Protocol Handler**
- [ ] Create IPFS service module
- [ ] Register `ipfs://` and `ipns://` protocol handlers
- [ ] Implement URL rewriting to gateway
- [ ] Test basic IPFS path resolution

**Phase 2: Configuration**
- [ ] Add `brave://settings/ipfs` page
- [ ] Allow custom gateway configuration
- [ ] Option to enable/disable IPFS support

**Phase 3: Enhanced Support (Optional)**
- [ ] Local node integration
- [ ] DHT connectivity
- [ ] Peer management

**Files to Create/Modify**:
- `brave/browser/ipfs/` — IPFS service
- `brave/browser/ui/webui/ipfs_settings/` — Settings UI
- `brave/browser/protocol_handler/` — IPFS protocol handler
- Build config: Add `enable_ipfs_support` flag

---

## Part 6: Extensibility & Plugin Architecture

### Design Goals

1. **Service Abstraction**: Allow swapping Brave services with custom implementations
2. **Configuration Flexibility**: Environment variables, config files, command-line flags
3. **Plugin System**: Possible future extension points for third-party services
4. **Backwards Compatibility**: Graceful degradation if custom services unavailable

### Service Provider Pattern

#### 1. Sync Service Provider

```cpp
// Abstract interface
class SyncServiceProvider {
  virtual ~SyncServiceProvider() = default;
  
  virtual bool IsEnabled() const = 0;
  virtual void SyncData(const SyncData& data) = 0;
  virtual SyncData RetrieveSyncData(const std::string& device_id) = 0;
  virtual void RegisterDevice(const DeviceInfo& info) = 0;
  virtual std::vector<DeviceInfo> GetRegisteredDevices() = 0;
};

// Implementation: Brave (default, can be disabled)
class BraveSyncServiceProvider : public SyncServiceProvider { ... };

// Implementation: Stub (no-op)
class StubSyncServiceProvider : public SyncServiceProvider { ... };

// Implementation: Custom (user-provided)
class CustomSyncServiceProvider : public SyncServiceProvider { 
  // Connects to user's own sync backend
  // Could be WebDAV, Nextcloud, custom REST API
};

// Factory
std::unique_ptr<SyncServiceProvider> CreateSyncServiceProvider(
    const std::string& provider_type,
    const SyncProviderConfig& config) {
  if (provider_type == "brave") return std::make_unique<BraveSyncServiceProvider>();
  if (provider_type == "custom") return std::make_unique<CustomSyncServiceProvider>(config);
  return std::make_unique<StubSyncServiceProvider>();  // Default
}
```

#### 2. AI Service Provider

```cpp
class AIServiceProvider {
  virtual ~AIServiceProvider() = default;
  
  virtual bool IsAvailable() const = 0;
  virtual std::string SendMessage(
      const std::string& message,
      const std::vector<std::string>& context,
      const std::string& model) = 0;
  virtual std::vector<std::string> GetAvailableModels() const = 0;
};

// Implementations: BraveAI, Ollama, OpenAI, etc.
// Configuration:
// --ai-provider=ollama --ai-config="http://localhost:11434"
// --ai-provider=openai --ai-config="sk-xxx"
// --ai-provider=none (default in unbraved)
```

#### 3. Wallet RPC Provider

```cpp
class WalletRPCProvider {
  virtual ~WalletRPCProvider() = default;
  
  virtual std::string SendRPC(
      const std::string& chain_id,
      const std::string& method,
      const std::string& params) = 0;
  virtual std::vector<std::string> GetSupportedChains() const = 0;
};

// Configuration via JSON file:
// {
//   "ethereum": "http://localhost:8545",
//   "solana": "http://localhost:8899"
// }
```

### Configuration System

**Priority (highest to lowest)**:
1. Command-line arguments `--service-provider=...`
2. Environment variables `UNBRAVED_SERVICE_PROVIDER_...`
3. Configuration file `~/.config/unbraved/services.json`
4. Built-in defaults (disabled for Brave services)

**Example Configuration File**:

```json
{
  "sync": {
    "enabled": false,
    "provider": "none"
  },
  "ai": {
    "enabled": false,
    "provider": "none"
  },
  "wallet_rpc": {
    "provider_type": "custom_endpoints",
    "endpoints": {
      "ethereum": "http://localhost:8545",
      "solana": "http://localhost:8899",
      "bitcoin": "http://my-bitcoin-node:18332"
    }
  },
  "news": {
    "enabled": false
  }
}
```

---

## Part 7: Migration Path & Implementation Roadmap

### Phase 1: Foundation (Weeks 1-4)

**Goals**: Remove highest-priority Brave features, disable non-essential services

**Tasks**:
1. Remove Brave Rewards entirely (not replaceable)
2. Remove Brave News (low complexity)
3. Remove Brave VPN (low complexity)
4. Disable Brave Leo (flag-based)
5. Disable Brave Wallet (flag-based)
6. Update branding (strings, icons, logos)
7. Test browser compilation and basic functionality

**Deliverables**:
- Clean build with Brave Rewards/News/VPN removed
- Rebranded UI (unbraved-chromium-meta)
- Feature flags for Leo/Wallet disabling

**Effort**: 3-4 weeks (1-2 developers)

### Phase 2: Core Services Abstraction (Weeks 5-12)

**Goals**: Abstract Sync and Accounts services, design extensibility

**Tasks**:
1. Design SyncServiceProvider interface
2. Implement StubSyncServiceProvider (no-op)
3. Make Brave accounts optional/local-only
4. Design authentication abstraction (if needed)
5. Create configuration system for service selection
6. Document extensibility architecture
7. Test with custom service implementations

**Deliverables**:
- Service provider abstractions for Sync/Accounts
- Configuration system working
- Documentation for custom service development

**Effort**: 6-8 weeks (2-3 developers)

### Phase 3: Optional Enhancements (Weeks 13+)

**Goals**: Add custom service implementations, restore IPFS, optimize

**Tasks**:
1. Implement custom sync provider (Nextcloud, WebDAV example)
2. Implement IPFS gateway support
3. Design/implement Wallet RPC abstraction
4. Optimize build size (remove unused Brave code)
5. Create deployment/build scripts
6. Develop documentation and howto guides
7. Community engagement and feedback

**Deliverables**:
- Working custom sync implementation
- IPFS gateway support
- Build/deployment automation

**Effort**: Ongoing (1-2 developers, community contributions)

### Testing Strategy

**Unit Tests**:
- Service provider factory tests
- Configuration parsing tests
- URL rewriting tests (for removed features)

**Integration Tests**:
- Feature flag disabling (ensure UI doesn't crash)
- Custom service provider loading
- Cross-platform builds (Windows, macOS, Linux)

**Manual Testing**:
- UI visual consistency after rebrand
- Feature removal doesn't break core browser
- Settings pages work correctly
- Sync/Account features gracefully disabled
- Performance impact minimal

---

## Part 8: Challenges & Considerations

### Technical Challenges

1. **Tight Chromium Integration**
   - Some features (Shields, Sync) are deeply embedded
   - Requires careful refactoring to abstract
   - Risk of regressions if not done carefully

2. **Duplicate Code Maintenance**
   - Keeping custom fork means maintaining merge conflicts with Brave upstream
   - Regular Chromium/Brave updates need to be tracked
   - Security patches must be applied promptly

3. **Configuration Complexity**
   - Many service endpoints, flags, and options
   - Risk of security issues if misconfigured
   - Documentation must be comprehensive

4. **Platform-Specific Code**
   - Sync, Accounts, VPN have Windows/macOS/Linux/Android/iOS-specific implementations
   - Each platform requires careful testing
   - Code paths may differ significantly

### Community & Maintenance

1. **Fork Sustainability**
   - Who maintains the codebase long-term?
   - How often are Chromium/Brave updates merged in?
   - Security review process established?

2. **Feature Parity**
   - Does unbraved-chromium-meta aim to keep all Chromium features?
   - Or selectively remove features like Ungoogled Chromium?
   - Regular decision-making needed

3. **Documentation**
   - Building from source requires substantial documentation
   - Configuration of custom services needs guides
   - Troubleshooting documentation essential

### Legal & Regulatory

1. **License Compliance**
   - Chromium (BSD), Brave (MPL-2)
   - Custom code should be compatible
   - Must include proper license headers/notices

2. **Trademark Issues**
   - Cannot use "Brave" branding
   - Must choose non-conflicting name
   - Legal review recommended for project name

3. **Third-Party Dependencies**
   - Wallet RPC providers (Chainstack, etc.)
   - AI model providers
   - May have usage terms/agreements

---

## Part 9: References & Resources

### Official Repositories
- **Brave Browser** (meta): https://github.com/brave/brave-browser
- **Brave Core** (source): https://github.com/brave/brave-core
- **Chromium**: https://chromium.googlesource.com/
- **Ungoogled Chromium**: https://github.com/ungoogled-software/ungoogled-chromium

### Documentation
- Brave Contributing: https://github.com/brave/brave-core/blob/master/CONTRIBUTING.md
- Brave Documentation: https://github.com/brave/brave-core/blob/master/docs/README.md
- Chromium Build Documentation: https://chromium.googlesource.com/chromium/src/+/main/docs/

### Related Projects
- **Ungoogled Chromium**: Feature-culled Chromium build
- **Librewolf**: Firefox derivative focused on privacy
- **Pale Moon**: Legacy browser maintenance
- **Tor Browser**: Firefox-based with Tor integration

### Custom Service Implementation References
- **Nextcloud**: Self-hosted cloud suite (for Sync replacement)
- **Ollama**: Local LLM runner (for AI replacement)
- **Infura/Alchemy**: Public Ethereum RPC (for Wallet replacement)

---

## Conclusion

Removing Brave-specific features from Brave Browser is technically feasible but requires substantial engineering effort. The most significant challenges are:

1. **Sync & Accounts**: Deeply integrated, require significant refactoring
2. **Ongoing Maintenance**: Keeping fork synced with Chromium/Brave upstream
3. **Testing**: Comprehensive cross-platform testing needed
4. **Documentation**: Clear guides for building, configuring, deploying

The **unbraved-chromium-meta** project is viable with:
- Clear prioritization of feature removal (Phase 1: Easy removals first)
- Proper service abstraction and plugin architecture (Phase 2: Core agility)
- Strong documentation and community support (Phase 3+: Long-term viability)
- Regular security updates and Chromium sync

**Recommended Project Scope**: Start with Phase 1 (foundation) to validate feasibility, then proceed to Phase 2-3 based on community interest and maintenance capacity.

---

**Document Version**: 1.0  
**Last Updated**: April 2026  
**Status**: Research Complete  
**Recommendations**: Proceed to Phase 1 implementation planning
