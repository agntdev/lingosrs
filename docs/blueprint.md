# SRS Language Learning Bot — Bot specification

**Archetype:** education

**Voice:** encouraging and concise — write every user-facing message, button label, error, and empty state in this voice.

A private Telegram bot implementing a spaced-repetition system for language learning. Users create and manage flashcard decks (word + translation + example), review cards with a 4-button rating system, and receive personalized scheduling and reminders. Sessions persist across devices, and all data remains private per user.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Individual language learners
- Mobile-first users
- Privacy-conscious learners

## Success criteria

- Users complete daily review sessions with due cards
- Users maintain consistent review streaks over weeks
- Users add and manage at least 3 decks per month

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu
- **/newdeck** (command, actor: user, command: /newdeck) — Create a new card deck
- **/add** (command, actor: user, command: /add) — Add a new flashcard to current deck
- **/decks** (command, actor: user, command: /decks) — List all decks with review stats
- **/stats** (command, actor: user, command: /stats) — Show learning progress and streaks
- **/resume** (command, actor: user, command: /resume) — Resume an interrupted review session
- **Start Reviews** (button, actor: user, callback: reviews:start) — Begin review session for all due cards
- **Import Starter Deck** (button, actor: user, callback: decks:import_starter) — Copy a read-only starter deck into private collection

## Flows

### Onboarding
_Trigger:_ /start

1. Greet user
2. Ask for daily new-card limit
3. Ask for daily reminder time
4. Offer starter decks

_Data touched:_ User

### Card Creation
_Trigger:_ /newdeck

1. Request deck name
2. Confirm deck creation
3. Prompt to add first card

_Data touched:_ Deck, User

### Card Addition
_Trigger:_ /add

1. Request front (word)
2. Request back (translation)
3. Request optional example sentence
4. Confirm card creation

_Data touched:_ Card, Deck

### Review Session
_Trigger:_ reviews:start or deck review

1. Build session from due cards and new cards
2. Show card front
3. User taps Reveal
4. Show card back + example
5. User selects rating (Again/Hard/Good/Easy)
6. Update SRS parameters
7. Show session summary

_Data touched:_ Card, Review session

### Session Resume
_Trigger:_ /resume

1. Load last session state
2. Continue from last card position

_Data touched:_ Review session

### Settings Update
_Trigger:_ settings:update

1. Show current settings
2. Prompt for changes
3. Save updated settings

_Data touched:_ User

### Notification Handling
_Trigger:_ scheduled notification

1. Send daily summary with due count
2. Offer quick 'Start Reviews' button
3. Send instant due-card reminder if enabled

_Data touched:_ User, Scheduled notification

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram account with private settings and progress
  - fields: Telegram ID, Time zone, Daily new-card limit, Reminder time, Notification preferences
- **Deck** _(retention: persistent)_ — Named collection of cards
  - fields: Name, Card count, New cards, Learning cards, Due cards, Total cards
- **Card** _(retention: persistent)_ — Flashcard with SRS metadata
  - fields: Front (word), Back (translation), Example sentence, Creation date, Last reviewed, Ease factor, Interval, Review history, Repetition count, Due date, Learning state
- **Review session** _(retention: session)_ — Active review sequence state
  - fields: Card queue, Current index, Session progress, Paused state
- **Scheduled notification** _(retention: persistent)_ — Reminder for due reviews
  - fields: Notification time, User ID, Due card count

## Integrations

- **Telegram** (required) — Bot API messaging and notifications
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure default new-card limit
- Set default reminder time
- Add/remove starter decks
- Enable/disable CSV import
- Configure notification templates

## Notifications

- Daily summary with due review count and 'Start Reviews' button
- Instant due-card reminder when card becomes due (optional)
- Session completion summary with streak update

## Permissions & privacy

- All user data is private and tied to Telegram account
- No public sharing or discovery of decks
- Data is not accessible by other users
- User can delete their account and data at any time

## Edge cases

- Empty deck: show guided message to add cards or import starter deck
- Finished session: show congratulatory message and suggest next steps
- Interrupted session: save state and allow resume
- No due cards: show friendly message with options to add new cards

## Required tests

- End-to-end review session with all 4 rating outcomes
- Session persistence across device changes
- Notification delivery at scheduled times
- CSV import of 100+ cards
- Starter deck import and editing
- Empty deck handling
- User settings update and retention

## Assumptions

- Using a standard SRS algorithm (ease factor + interval multipliers) for card scheduling
- Default daily new-card limit of 10
- Default reminder time of 20:00 local time
- Private 1:1 Telegram messages only for all interactions
- Starter decks include 100 high-frequency words example
