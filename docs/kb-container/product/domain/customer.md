# Customer & Contact Domain Concept

**Last Updated**: April 2026

## Context

The customer hierarchy has two levels: the `Customer` (company) and the `Contact`/`End User` (individual person at the company).

## Customer

A `Customer` is a business entity that the MSP services:
- `id` — numeric customer ID
- `name` — company name
- Has many contacts, assets, and tickets

## Contact / End User

A `Contact` (also referred to as `End User`) is an individual person at a customer company:
- `id` — numeric contact ID
- First name, last name, email, phone
- Belongs to a `Customer`
- May be associated with tickets as the reporting party

## Flutter Naming Inconsistency

The codebase uses both `customers` and `end_user` features. The `CustomersRepository` handles the company-level data, while `EndUsersRepository` handles individual contacts. The Syncro backend API may use different terminology — always check the API contract.

## Flutter Features

- `customers` feature — search customers, view customer detail
- `end_user` feature — view contact detail, edit contact info
