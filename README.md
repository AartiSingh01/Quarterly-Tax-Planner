# Quarterly-Tax-Planner
Quarterly Tax Planner Plugin
Technical Walkthrough: Custom WordPress Tax Calculator Plugin

Project: BizTax Playbook — Quarterly Tax Planner
URL: biztaxplaybook.com (in final development)
Role: Sole developer — architecture, backend, frontend, and client delivery

The Problem

A client needed a WordPress-based quarterly tax calculator for self-employed business owners and S-Corp owners. The requirements were complex:

Handle multiple income types (SE income, S-Corp distributions, spouse W-2 wages)
Calculate SE tax, QBID with SSTB phase-out rules, safe harbor comparisons, and Additional Medicare Tax
Be fully maintainable by a non-technical client every year when IRS figures change — without developer involvement
Deliver real-time results without page reload

No off-the-shelf plugin could handle this logic. I designed and built a custom solution from scratch.

Architecture Decision

I chose a three-layer architecture to keep data, logic, and display completely separate — making the system maintainable, testable, and expandable.

Layer 1 — Data (JetEngine Custom Post Type)
Created a custom post type (tax_year) where all annual IRS figures are stored: tax brackets, standard deductions, QBI thresholds, SE rates, and safe harbor percentages. One post per tax year. The client adds 2027 data each October by filling in a form — no code changes required.

Layer 2 — Calculation (Custom PHP Plugin)
Built a modular PHP plugin with four classes:

BizTax_Calculator_Engine — orchestrates the full calculation flow
BizTax_Tax_Data — the only class that queries JetEngine, with hardcoded IRS fallbacks so nothing breaks if data is missing
BizTax_QBID — handles per-source QBID calculation with SSTB phase-out and wage limit logic
ajax-handler.php — registers REST API and admin-ajax endpoints with nonce verification and full input sanitization

Layer 3 — Frontend (JetFormBuilder + Custom JavaScript)
A multi-step form with conditional field visibility, custom validation, and a manual Calculate button. Results appear without page reload. Rows with zero values are automatically hidden so the output stays clean.

Technical Challenges I Solved

Challenge 1 — JetFormBuilder/Elementor Script Conflict

JetFormBuilder’s JavaScript was not initializing due to a conflict with Elementor’s performance optimizations. The form submit button did nothing.

How I solved it: I intercepted the form submit event at the JavaScript capture phase — before JFB’s broken handler could fire — and bypassed its AJAX pipeline entirely. I sent the form data directly to my custom REST API endpoint instead. This gave me full control over the request, response, and error handling.

Challenge 2 — Inverted QBID Phase-Out Bug

The QBID deduction was showing approximately 2% of the correct value for incomes just above the phase-out threshold, instead of approximately 98%.

How I solved it: I systematically audited the phase-out calculation against IRS Publication 535 formulas and found an inverted percentage — the code was calculating phase_out_ratio instead of 1 - phase_out_ratio. Fixing this one line corrected dollar discrepancies of thousands of dollars in the output.

Challenge 3 — PHP Boolean Coercion Bug

SE and S-Corp income was annualizing even when those income types were deselected by the user.

How I solved it: The form was passing the string 'no' as a flag value. In PHP, any non-empty string evaluates as true in a boolean context — so if ($is_se_income) was always true. I fixed this by explicitly checking $is_se_income === 'yes' instead of truthy evaluation throughout the calculation engine.

Challenge 4 — Stale Nonce Caching

The REST API was returning nonce verification failures intermittently because the caching plugin was serving stale nonces to returning visitors.

How I solved it: Added server-side no-cache headers specifically for the calculator page and excluded it from all caching layers in the hosting configuration. Nonce failures dropped to zero.

Deliverables
Production-ready WordPress plugin (ZIP) installable on any WordPress site
JetEngine CPT with 2024, 2025, and 2026 IRS data entered and verified
JetFormBuilder multi-step form with all fields, conditions, and validation configured
Client-facing documentation (Word doc) covering how to update IRS data annually and troubleshoot common issues
Developer documentation covering architecture, file structure, and expansion roadmap
Stack

WordPress · PHP 7.4+ · JetEngine CPT · JetFormBuilder · Elementor · Vanilla JavaScript · WordPress REST API · CSS
