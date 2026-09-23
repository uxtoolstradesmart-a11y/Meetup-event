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

The prototype markup has these values typed in; the `BOOKING` object shows the shape the real data should take. 

## Interaction spec

| Behaviour | Detail |
|---|---|
| Entrance | Drops in from above: translateY(-170px) rotateX(78deg) → settle with overshoot, 1.3s. "CONFIRMED" stamp slams in at 1.5s |
| Idle | Gentle 3D sway (rotateY ±9°, rotateX ±4°, 6s loop); holographic foil strip drifts; floor shadow breathes |
| Depth | Card has 5 edge layers (visible thickness) and content layers lifted with translateZ (title 34px, booking ID 26px, date/venue 22px) |
| No manual rotation | The ticket can't be dragged, spun or flipped; only the gyro tilt moves it. It's a static `role="img"` element with a full-text `aria-label` (no back face / QR on this screen). Booking ID is set small (14px Roboto Mono) |
| Actions | Add to calendar → turns green "Added" + toast; Invite a trader → toast "Invite link copied" |
| Gyro parallax | Tilting the phone tilts the ticket via a `#gyro` layer wrapped around `#card3d` (so it adds to drag/flip). Max ±13° plus up to ~9px of shift, gain 1°/° of device tilt, eased 0.14 per frame. The neutral angle slowly re-centres (0.2% per event). Falls back to the accelerometer gravity vector on phones without a gyroscope. Add `?debug` to the URL for a sensor readout, so it works however the phone is held. Glare and foil follow the tilt; the floor shadow slides the other way. While gyro is active the idle sway stops. Landscape is handled via `screen.orientation.angle` |
| Motion permission | Listeners start on load. Android (Samsung Internet, Chrome) needs no tap: where Chrome exposes `requestPermission()` it is called silently on load and grants without a gesture. iOS 13+ needs one tap the first time: any tap on the page asks (document-level `click`, not `pointerdown`), and an "Enable tilt effect" chip appears only if no motion data arrived within 0.9s. **In the Angular app, call `DeviceOrientationEvent.requestPermission()` inside the Register/Pay button's click handler** so iOS users grant it there and the ticket tilts the moment the confirmation page opens |
| Haptics | `buzz()` uses `navigator.vibrate` on Android. iOS Safari has no vibrate API, so iOS 18+ gets a system tick by toggling a hidden native `<input type="checkbox" switch>`. The celebration pattern `[8, 660, 22, 150, 8, 640, 14, 280, 6, 140, 6]` (starting at 300ms) lines up with the check pop, the ticket landing and bouncing, the CONFIRMED stamp, and the confetti. Flip: `[12,50,6]`; snap back to the same face: 6ms; Add to calendar: `[10,60,10]`; Invite: 10ms |
| Haptics caveat | Browsers only allow haptics after the user has interacted with the page. In the Angular app the confirmation is a route change after the user taps Register/Pay, so it's the same document and the celebration haptic fires. On a cold page load (like opening this prototype from a link) the first celebration can be silently blocked; the ↻ Replay button in the header replays the entrance, confetti and haptics |
| Reduced motion | All animation/transition off under `prefers-reduced-motion: reduce`; haptics off, tilt kept at half strength (it only moves when the user moves the phone) |

## Prompt to paste into Claude Code

> I'm attaching `confirmed-3d-ticket.html`, a working prototype of our Traders Meetup registration-confirmed screen with an interactive 3D ticket. Port it into our Angular app as a standalone `EventTicketComponent` (plus a `RegistrationConfirmedPage`). Keep the visual design and every interaction in HANDOFF.md identical. Requirements:
> 1. Take the booking as an `@Input()` typed interface matching the `BOOKING` object; no hard-coded values in the template.
> 2. Move the styles into the component SCSS; keep Roboto / Roboto Mono and the existing colour values.
> 3. Rewrite the pointer logic with Angular `HostListener`s or `fromEvent` and run it outside the Angular zone (`NgZone.runOutsideAngular`) so the tilt doesn't trigger change detection on every move; use `requestAnimationFrame` for transform updates.
> > 5. Wire "Add to calendar" to download an `.ics` file for the event and "Invite a trader" to the Web Share API with a clipboard fallback.
> 6. Respect `prefers-reduced-motion`, keep the ticket's descriptive `aria-label`, and keep touch targets ≥44px.
> 7. Test on iOS Safari and Android Chrome: check `backface-visibility`, `preserve-3d` and `touch-action: none` behave, and that dragging the ticket doesn't scroll the page.
> Show me the component tree first, then build it.
