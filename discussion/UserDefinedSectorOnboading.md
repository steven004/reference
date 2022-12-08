Author: Venus Team

## Summary
To enable Sealing as a Service, we need to leave enough time for sealed data move, for either online transferring or offline transport. In this discussion, we propose new sector proving methods separate the proveCommit and power onboarding to allow longer buffer for non-onboarding data moving.

## Motivation
Sealing as a service has grown from [an idea](https://filecoinproject.slack.com/archives/C0320QCV8SY/p1644244480718559) summarized in [a Google doc](https://docs.google.com/document/d/1zF1F6Q_Ya6IOUe9nfDOa0bNifvIa0TY4nR89omxy0kQ/edit) to one of the priorities among Filecoin data-onboarding roadmap. There are many potential merits of SaaS which includes lowering entry to Filecoin network, further separation of different roles, and the possibility of creating [new line of business](https://github.com/filecoin-project/lotus/discussions/9079).

When both Lotus and Venus team are working on the implementation of APIs and processes to support SaaS, there is one thing to be solved before the real business can be supported -  the duration is too short for a on-chain proved sector to be moved from one place to another. For any active sector, the SP has to do WindowedPoSt within 24 hours or less, or it will be set as fault. Usually, one day is not enough for SaaS to move data from the sealing service provider to an SP.

## Design
The problem is that every proveCommitted sector is put into a partition of a particular deadline, which will be checked periodically if WindowedPoSt is done in a particular timeframe. All sectors will be scanned every 24 hours.

This proposal is to separate the proveCommit and deadline assignment of a sector into different methods, so that the sealing service provider could generate a proof of a sector and put it on chain, but not assign a deadline for it, after the data is transferred to the SP, the SP can invoke another method to add the sector into a deadline.

## Discussion
This should be no problem for CC sectors to arbitrary buffer between the PoRep and first PoSt, however, for a sector having deals, when a deal is activated, the corresponding sector should be available, i.e. pass the WindowedPoSt, or there will be some slashes. (should this be handled by Market?)

Any other better solutions to discuss?


## Consideration
The implementation of this proposal will be simply add two more methods to the miner actor (?), so there is no impact to the current system, in other words, it is fully back-compatible.

You are welcome for any comments.
