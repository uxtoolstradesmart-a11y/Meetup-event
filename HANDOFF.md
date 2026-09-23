# Traders Meetup 2026 — 3D ticket handoff

`confirmed-3d-ticket.html` is a self-contained prototype of the confirmation screen.
Open it in a browser — no build, no dependencies. All logic is plain JS at the bottom of the file.

## Dummy data (replace from the registration/payment API)

```js
const BOOKING = {
  id: 'TSM-26-04821',
  attendee: 'Aarav Sharma',
  email: 'aarav.sharma@example.com',
  date: 'Sat, 14 Nov 2026',
  time: '2:00 – 6:30 PM',
  venue: 'Meridian Hall, Lower Parel',
  city: 'Mumbai',
  daysToGo: 52
};
```

The prototype markup has these values typed in; the `BOOKING` object shows the shape the real data should take. The QR code is a decorative pattern — generate a real one from the booking ID.

## Interaction spec

| Behaviour | Detail |
|---|---|
| Entrance | Drops in from above: translateY(-170px) rotateX(78deg) → settle with overshoot, 1.3s. "CONFIRMED" stamp slams in at 1.5s |
| Idle | Gentle 3D sway (rotateY ±9°, rotateX ±4°, 6s loop); holographic foil strip drifts; floor shadow breathes |
| Hover / finger move | Tilt toward pointer: rotateY ±17°, rotateX ±13°; glare follows pointer; foil shifts with pointer |
| Drag | Horizontal drag spins freely (0.75° per px); on release snaps to nearest face with spring `cubic-bezier(.2,1.5,.35,1)` 0.9s |
| Tap | Flips 180° (front ticket ↔ back entry pass with QR). Every face change fires a sparkle burst |
| Keyboard | Enter/Space on the ticket button flips it |
| Depth | Card has 5 edge layers (visible thickness) and content layers lifted with translateZ (title 34px, booking ID 26px, date/venue 22px, QR 30px) |
| Actions | Add to calendar → turns green "Added" + toast; Invite a trader → toast "Invite link copied" |
| Reduced motion | All animation/transition off under `prefers-reduced-motion: reduce` |

## Prompt to paste into Claude Code

> I'm attaching `confirmed-3d-ticket.html`, a working prototype of our Traders Meetup registration-confirmed screen with an interactive 3D ticket. Port it into our Angular app as a standalone `EventTicketComponent` (plus a `RegistrationConfirmedPage`). Keep the visual design and every interaction in HANDOFF.md identical. Requirements:
> 1. Take the booking as an `@Input()` typed interface matching the `BOOKING` object; no hard-coded values in the template.
> 2. Move the styles into the component SCSS; keep Roboto / Roboto Mono and the existing colour values.
> 3. Rewrite the pointer logic with Angular `HostListener`s or `fromEvent` and run it outside the Angular zone (`NgZone.runOutsideAngular`) so the tilt doesn't trigger change detection on every move; use `requestAnimationFrame` for transform updates.
> 4. Replace the decorative QR with a real QR generated from the booking ID (suggest a lightweight library).
> 5. Wire "Add to calendar" to download an `.ics` file for the event and "Invite a trader" to the Web Share API with a clipboard fallback.
> 6. Respect `prefers-reduced-motion`, keep the ticket a real `<button>` with an updating `aria-label`, and keep touch targets ≥44px.
> 7. Test on iOS Safari and Android Chrome: check `backface-visibility`, `preserve-3d` and `touch-action: none` behave, and that dragging the ticket doesn't scroll the page.
> Show me the component tree first, then build it.
