# Public records

Techniques using government and published datasets.

- [Healthcare providers (NPI)](#healthcare-providers-npi)
- [PPP loans](#ppp-loans)
- [Property ownership](#property-ownership)
- [License plate to VIN](#license-plate-to-vin)

---

## Healthcare providers (NPI)

Every US healthcare provider who bills insurance has a public **National Provider Identifier** record, with specialty and practice addresses.

- Search by name at [npiregistry.cms.hhs.gov](https://npiregistry.cms.hhs.gov).
- API (JSON):
  ```
  https://npiregistry.cms.hhs.gov/api/?version=2.1&first_name=Jane&last_name=Doe
  ```
- Narrow common names by state, city or specialty.
- For license status and discipline, use the state medical board's own lookup.

---

## PPP loans

The SBA published every Paycheck Protection Program loan from the COVID period, and ProPublica made them searchable.

1. Search the business at [projects.propublica.org/coronavirus/bailouts](https://projects.propublica.org/coronavirus/bailouts/) (or federalpay.org).
2. Match by city, state and amount — companies often had more than one loan.
3. The loan page shows jobs reported, lender, approval date and forgiveness amount.

---

## Property ownership

Famous owners rarely appear on deeds under their public name — they typically buy through trusts or LLCs.

1. **Search the address directly**, in quotes. Real estate news and listing sites sometimes name the owner.
2. **Listing history** on Zillow, Redfin or Compass gives sale dates, prices and MLS numbers. Search news for the town plus the sale price and year.
3. **County records:** assessor portals show parcels and sale history; owner names availability varies by county.
4. **LLCs:** look up the entity at the state's business registry (in California, [bizfileonline.sos.ca.gov](https://bizfileonline.sos.ca.gov)) for its agent or manager.
5. **Use every hint:** the challenge title, and answer-format notes like "not the same spelling as the name on file" (records often mangle names — submit the correct public spelling).
6. **Confirm** by matching listing photos or satellite view against photos in a news story.

> Be careful publishing what you find: tying a named person to their home address is doxxing, even when every piece is technically public.

---

## License plate to VIN

A VIN identifies the vehicle (make, model, year, factory), not the owner. Owner details from DMV records are protected in the US by the Driver's Privacy Protection Act.

1. On **[FaxVIN](https://www.faxvin.com)**, search by license plate, pick the state and enter the plate. It returns the VIN and basic vehicle details.
2. Confirm with the **[NHTSA VIN decoder](https://vpic.nhtsa.dot.gov/decoder/)** — make, model and year should match.
3. Some states have their own vehicle checks by VIN (e.g. Florida's FLHSMV) for title status.

- A VIN is 17 characters and never contains I, O or Q — useful for spotting misreads.
- Vanity plates get reissued. If several vehicles come back, use the most recent.
