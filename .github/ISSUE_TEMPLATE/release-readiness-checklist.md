---
name: Release Readiness Checklist
about: Checklist to Assist Product Team in determining the Release Readiness of a product release.
title: RRC - Product and Version
labels: release
assignees: ''

---

# Release Readiness Checklist (Completed by the Product Team!)

## Description

* The Release Readiness Checklist should be completed by the product team prior to each release.  
* This checklist will assist the team in determining if the release is ready for production deployment.  
* **This checklist should be added to each product production release milestone and completed.**
* If the release does not meet some of the items in the checklist, then simply leave them unchecked and add comments to the Release Readiness Checklist providing justification as to why the release should be deployed even though not all release criteria have not been met.
* If the item is NA, simply add 'NA' to the end of the RRC Item.

## Releases/Version Descriptions
Insert links to the  product component releases that are part of this product release.  The releases (tags) are in the product's components' code GitHub repositories.

## Release Readiness Checklist
If an item is NA per the [Release User Stories](https://vaww.vaco.portal.va.gov/sites/OIT/epmo/TRS/RRO%20%20Release%20Public%20Artifacts/Release%20Compliance%20User%20Stories.pdf), please specify NA.

- [ ] Links to Product Code Releases
	URLs:
- [ ] User Stories (US) are completed and approved
- [ ] US Tests are created
- [ ] US Tests traced to Release User Stories
- [ ] All non-production testing is complete (functional, regression, integration, SQA, UAT, etc.)
- [ ] No open critical or high severity defects being implemented into production
- [ ] All open high exposure risks are mitigated
- [ ] AFMP completed and approved
- [ ] If applicable, IOC site Concurrence
- [ ] If applicable HPS AC Concurrence
- [ ] Architecture verified
- [ ] Section 508 addressed
- [ ] Valid ATO
- [ ] Deployment, Installation, Rollback and Backout procedures completed
- [ ] Operations Procedures completed
- [ ] Operations RACI completed
- [ ] SLAs/OLAs completed and approved
- [ ] User Documentation complete
