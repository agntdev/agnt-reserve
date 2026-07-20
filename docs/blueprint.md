# TableReserve Bot — Bot specification

**Archetype:** booking

**Voice:** friendly and helpful — write every user-facing message, button label, error, and empty state in this voice.

A restaurant table reservation bot that manages bookings, prevents double-bookings, and sends reminders. Guests can reserve tables, reschedule, or cancel via inline buttons. Owners get a dashboard with real-time availability, booking lists, and no-show tracking.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Restaurant guests
- Restaurant owners/managers

## Success criteria

- Guests can make and manage reservations without errors
- Owner can view and manage all bookings in real-time
- No double-bookings or overcapacity situations occur

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu for guests or owner authentication
- **Make a reservation** (button, actor: user, callback: booking:start) — Initiates the reservation flow for guests
  - inputs: date, party size, time slot, guest info
  - outputs: booking confirmation, booking code, reminder
- **Owner Login** (button, actor: user, callback: owner:login) — Initiates owner authentication flow
  - inputs: owner credentials
  - outputs: owner dashboard, booking management tools

## Flows

### Guest Reservation Flow
_Trigger:_ /start or 'Make a reservation' button

1. Greeting and language selection
2. Date selection (calendar picker)
3. Party size selection
4. Available time slot selection
5. Guest info collection (optional)
6. Booking confirmation with code
7. Reminder scheduling

_Data touched:_ booking, restaurant settings, tables

### Owner Dashboard Flow
_Trigger:_ Owner Login

1. Owner authentication
2. Display today's bookings summary
3. Display upcoming bookings
4. Display no-shows
5. Booking management actions

_Data touched:_ bookings, restaurant settings, tables

### Reschedule Flow
_Trigger:_ Reschedule button from booking confirmation or reminder

1. Date selection
2. Party size confirmation
3. Available time slot selection
4. Booking update confirmation

_Data touched:_ booking, restaurant settings, tables

### Reminder Flow
_Trigger:_ Scheduled reminder event

1. Send reminder message
2. Include reschedule/cancel buttons

_Data touched:_ booking, reminder state

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Restaurant Settings** _(retention: persistent)_ — Restaurant configuration parameters
  - fields: opening hours (per weekday), seating duration, slot granularity, advance booking window, time zone, table combining rules
- **Tables** _(retention: persistent)_ — List of tables with capacities
  - fields: table ID, capacity, is_combinable
- **Booking** _(retention: persistent)_ — Reservation details
  - fields: guest name, phone number, party size, date, time slot, assigned table(s), booking code, status, created_at, reminder_sent
- **Owner Account** _(retention: persistent)_ — Owner access credentials and preferences
  - fields: chat ID, notification preferences, dashboard settings

## Integrations

- **Telegram** (required) — Bot API messaging
- **Internal Scheduler** (required) — Send reminders at scheduled times
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- View today's bookings
- View upcoming bookings
- Mark booking as arrived/no-show
- Reschedule booking
- Cancel booking
- Configure restaurant settings
- Adjust reminder time
- Adjust seating duration
- Adjust advance booking window
- Enable/disable table combining
- Set time zone
- Configure notification preferences

## Notifications

- Guests receive booking confirmation
- Guests receive reminder before reservation
- Owner receives new booking notification
- Owner receives daily digest (optional)
- Owner receives no-show alerts

## Permissions & privacy

- Guest contact info stored privately and only shown to owner and guest
- Booking codes are unique and non-identifiable
- Owner can only access their own restaurant's data
- Guests can only manage their own bookings

## Edge cases

- Guest tries to book outside opening hours
- Guest tries to book with party size exceeding available tables
- Multiple guests try to book same time slot simultaneously
- Owner tries to access another restaurant's data
- Guest provides invalid phone number format
- Guest cancels after reminder has been sent

## Required tests

- End-to-end reservation flow with double-booking prevention
- Owner dashboard displays accurate real-time data
- Reminder scheduling and delivery works correctly
- Reschedule flow updates availability correctly
- Guest info collection handles optional fields gracefully
- Table combining rules work as configured

## Assumptions

- Restaurant settings are pre-configured by owner during setup
- Owner has access to Telegram and can manage the bot
- Guests have basic Telegram literacy
- Internal scheduler is reliable and persistent
