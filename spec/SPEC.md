# Sri Lanka NIC Validator Specification

## 1. Scope

This library provides mathematical and logical validation for Sri Lankan National Identity Card (NIC) numbers.
**Disclaimer:** This library validates format, checksums, and logical consistency. It does **not** query the Department for Registration of Persons (DRP) database and must not be used as the sole method for legal KYC or identity verification.

## 2. Input Contract

The validator accepts:

- `nic` (String): The NIC number (e.g., "901234567V" or "199012345678").
- `dob` (String): Date of Birth in ISO-8601 format (e.g., "1990-05-03").
- `gender` (String/Enum): "MALE" or "FEMALE".

## 3. Validation Rules

### 3.1 Format & Checksum

**Old Format (9 Digits + 1 Letter):**

- Regex: `^\d{9}[VX]$`
- Weights for first 8 digits: `[8, 7, 6, 5, 4, 3, 2, 1]`
- Math: `Sum = Σ(digit * weight)`. `Remainder = Sum % 11`.
- Check Letter Map: `0=A, 1=B, 2=C, 3=D, 4=E, 5=F, 6=G, 7=H, 8=I, 9=J, 10=V`.

**New Format (12 Digits):**

- Regex: `^\d{12}$`
- Weights for first 11 digits: `[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]`
- Math: `Sum = Σ(digit * weight)`. `Remainder = Sum % 11`.
- Check Digit: `11 - Remainder`. If result is `10`, digit is `X`. If result is `11` (remainder 0), digit is `0`.

### 3.2 Century Logic (Old Format Only)

- If the first two digits are `00` to `15`, the birth year is `2000` to `2015`.
- If the first two digits are `16` to `99`, the birth year is `1916` to `1999`.

### 3.3 Gender & Day-of-Year Logic

- Extract the 3-digit day-of-year sequence (digits 3-5 for Old, 5-7 for New).
- If value > 500: Gender is **FEMALE**, and `actualDayOfYear = value - 500`.
- If value ≤ 500: Gender is **MALE**, and `actualDayOfYear = value`.

### 3.4 Triplet Validation

The extracted `year`, `actualDayOfYear`, and `inferredGender` must exactly match the provided `dob` and `gender` inputs.

- The provided `dob` must be a valid calendar date (e.g., Feb 29 is only valid in leap years).

## 4. Configuration (Global)

The library supports a global configuration object:

- `minAge` (int, default: 18): Minimum allowed age.
- `maxAge` (int, default: 80): Maximum allowed age.
- `lenientMode` (bool, default: true): If true, automatically `.trim()` and `.toUpperCase()` the input NIC.
- `globalSalt` (string, required for hashing): A cryptographically random string (min 32 chars) for secure hashing.

## 5. Output Contract

Returns a `ValidationResult` object containing:

- `isValid` (bool): True only if format, checksum, triplet, and age rules all pass.
- `errors` (List[String]): Human-readable error messages (e.g., "Gender mismatch").
- `extracted` (Object):
  - `format` (String): "OLD" or "NEW"
  - `year` (int)
  - `dayOfYear` (int, 1-indexed, where 1 = Jan 1st)
  - `inferredGender` (String): "MALE" or "FEMALE"

## 6. Hashing Utility

- Method: `hash(nic, globalSalt)`
- Algorithm: SHA-256
- Logic: `SHA256(nic + globalSalt)`
- Purpose: Provides a deterministic, salted hash for safe database storage and uniqueness checks, complying with data privacy best practices.
