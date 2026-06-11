# 🇱🇰 Sri Lanka NIC Validator

A robust, framework-agnostic library for validating Sri Lankan National Identity Card (NIC) numbers. Built with a **Contract-First** approach to ensure mathematical accuracy, security, and seamless integration into both PHP (Laravel) and Java (Spring Boot) ecosystems.

## ✨ Features

- **Dual Format Support:** Validates both legacy (9-digit + V/X) and modern (12-digit) formats.
- **Triplet Validation:** Cross-checks NIC mathematical extraction against provided Date of Birth and Gender to prevent lazy fraud and typos.
- **Raw Data Extraction:** Returns parsed `year`, `dayOfYear`, and `inferredGender` without bloated date-library dependencies.
- **Secure Hashing:** Built-in SHA-256 salting utility for PDPA-compliant database storage.
- **Lenient Mode:** Auto-corrects common user input errors (trailing spaces, lowercase letters).

## ⚠️ Disclaimer

This library performs mathematical and logical validation. It does **not** query the Department for Registration of Persons (DRP) database. It should be used as a first-line validation tool, not as a sole method for legal KYC.

## 📦 Installation & Usage

*(Implementation details for Java and PHP will be added here as the `implementations/` folders are populated).*

### Example: Extracting Data (1-Liner)

```java
// Java Example
NicValidationResult result = validator.validate(nic, dob, gender);
if (result.isValid()) {
    LocalDate actualDob = LocalDate.ofYearDay(result.getExtracted().getYear(), result.getExtracted().getDayOfYear());
}
