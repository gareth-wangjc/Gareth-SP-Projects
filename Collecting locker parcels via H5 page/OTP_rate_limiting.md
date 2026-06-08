# OTP Rate Limiting — H5 Locker Collection

## The problem
The current OTP input accepts unlimited attempts. A 4-digit OTP has only 10,000 combinations, making it vulnerable to brute force if no attempt ceiling is enforced.

---

## What to implement

### Frontend (HTML changes)

**1. Add attempt tracking**

Add a counter variable near the top of your script block:

```js
let otpAttempts = 0;
const OTP_MAX_ATTEMPTS = 30;
const OTP_LOCKOUT_SECONDS = 300; // 5 minutes
```

---

**2. Wrap OTP submission logic with attempt check**

In your `otpSuccess()` or equivalent OTP validation function, add attempt handling before processing:

```js
function handleOTPSubmit() {
  const code = Array.from(document.querySelectorAll('.otp-input'))
    .map(i => i.value).join('');

  if (code.length < 4) return;

  otpAttempts++;

  if (otpAttempts > OTP_MAX_ATTEMPTS) {
    showOTPLockout();
    return;
  }

  // existing OTP verification logic here
  // if OTP is wrong, show remaining attempts warning
  const remaining = OTP_MAX_ATTEMPTS - otpAttempts;
  if (remaining <= 5) {
    showAttemptsWarning(remaining);
  }
}
```

---

**3. Add a lockout state to the OTP screen**

Below your existing OTP input block, add a hidden lockout message:

```html
<!-- Add inside your OTP step div -->
<div id="otpLockout" class="hidden flex flex-col items-center text-center px-4 mt-4">
  <div class="w-14 h-14 rounded-full bg-red-50 flex items-center justify-center mb-4">
    <svg width="28" height="28" viewBox="0 0 28 28" fill="none">
      <circle cx="14" cy="14" r="12" stroke="#ee4d2d" stroke-width="2"/>
      <path d="M14 8v6" stroke="#ee4d2d" stroke-width="2.5" stroke-linecap="round"/>
      <circle cx="14" cy="19" r="1.5" fill="#ee4d2d"/>
    </svg>
  </div>
  <p class="text-sm font-black text-slate-800 mb-1">Too many attempts</p>
  <p class="text-xs text-slate-400 leading-relaxed">
    You've entered the wrong OTP too many times.<br>Please try again in <span id="lockoutCountdown" class="font-bold text-slate-600">5:00</span>.
  </p>
</div>

<!-- Attempts warning (shown when ≤2 attempts left) -->
<div id="otpAttemptsWarning" class="hidden mt-3 text-center">
  <p class="text-xs text-red-500 font-semibold" id="attemptsWarningText"></p>
</div>
```

---

**4. Add lockout display and countdown logic**

```js
function showOTPLockout() {
  // Hide OTP inputs and submit button
  document.getElementById('otp-container').classList.add('hidden');
  document.getElementById('otpLockout').classList.remove('hidden');

  // Disable any resend OTP button if present
  const resendBtn = document.getElementById('btnResendOTP');
  if (resendBtn) resendBtn.disabled = true;

  // Start countdown
  let seconds = OTP_LOCKOUT_SECONDS;
  const countdownEl = document.getElementById('lockoutCountdown');

  const interval = setInterval(() => {
    seconds--;
    const m = Math.floor(seconds / 60);
    const s = seconds % 60;
    countdownEl.textContent = `${m}:${s.toString().padStart(2, '0')}`;

    if (seconds <= 0) {
      clearInterval(interval);
      resetOTPLockout();
    }
  }, 1000);
}

function resetOTPLockout() {
  otpAttempts = 0;
  document.getElementById('otpLockout').classList.add('hidden');
  document.getElementById('otp-container').classList.remove('hidden');
  // Clear OTP fields
  document.querySelectorAll('.otp-input').forEach(i => i.value = '');
  document.querySelectorAll('.otp-input')[0].focus();
}

function showAttemptsWarning(remaining) {
  const el = document.getElementById('otpAttemptsWarning');
  const text = document.getElementById('attemptsWarningText');
  if (!el || !text) return;
  text.textContent = `Incorrect OTP. ${remaining} attempt${remaining > 1 ? 's' : ''} remaining.`;
  el.classList.remove('hidden');
}
```

---

### Backend (for eng handoff)

The frontend limiting above is a UX guardrail only — it can be bypassed by a determined bad actor. Backend rate limiting is the authoritative control.

Flag for engineering:
- Rate limit OTP verification endpoint to **30 attempts per session token**
- On 30th failed attempt, invalidate the session token entirely (force re-scan)
- Return a specific error code (e.g. `OTP_MAX_ATTEMPTS_EXCEEDED`) so the frontend can show the lockout state
- Log all failed OTP attempts with session token, timestamp, and locker point ID for ops monitoring

---

## UX copy

| State | Copy |
|---|---|
| 1 attempt remaining | "Incorrect OTP. 1 attempt remaining." |
| 2 attempts remaining | "Incorrect OTP. 2 attempts remaining." |
| Locked out | "Too many attempts. Please try again in 5:00." |
| Lockout expired | *(OTP input reappears, counter resets)* |

---

## Notes
- Lockout duration (5 min) and max attempts (30) should be **BE configurable** per market if needed
- 30 attempts was chosen to avoid false lockouts for genuine users while still capping brute force exposure. At 30/10,000 combinations, a targeted attacker has a 0.3% success chance per session — acceptable given that physical locker presence is also required
- This is a frontend-only implementation until backend rate limiting is built; treat it as a UX layer, not a security guarantee
- Consider logging frontend lockout events to your analytics pipeline for visibility
