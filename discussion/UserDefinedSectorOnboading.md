Author: Venus Team

## Simple Summary

Allow deadline assignment of a sealed sector.

## Abstract

Given the limitations on current state of network infrastructure across the globe,  in order to achieve any meaningful Sealing-as-a-Service in scale, we need to leave enough time to move sealed sector from a SaaS provider to SPs through either online transferring or offline transport. In this discussion, we propose new sector proving methods that separate sector proveCommit and on-chain sector power activation to allow longer buffer time for data movement between SaaS provider and SPs.

## Motivation

Sealing-as-a-Service has grown from [an idea](https://filecoinproject.slack.com/archives/C0320QCV8SY/p1644244480718559) summarized in [a Google doc](https://docs.google.com/document/d/1zF1F6Q_Ya6IOUe9nfDOa0bNifvIa0TY4nR89omxy0kQ/edit) to one of the priorities among Filecoin data-onboarding roadmap. There are many potential merits of SaaS which includes lowering entry to Filecoin network, further separation of roles, and the possibility of creating [new line of business](https://github.com/filecoin-project/lotus/discussions/9079).

As both Lotus and Venus team are working on [SaaS APIs designs](https://github.com/filecoin-project/lotus/discussions/9079) and make adjustments to support SaaS, there is one thing left in the puzzle to be solved before SaaS business can be done in scale -  the time window is too short for a on-chain proved sector to be moved from one place to another. For any active sector, the SP has to do WindowedPoSt within 24 hours or less, or it will be declared as faulty. It is evident that 24 hour is not practical for sectors to be moved from a SaaS provider to an SP.

Currently, there is a 30-day buffer between the preCommit and proveCommit of a particular sector. The sealing provider could hold and send the proveCommit message until the data is transferred to her client successfully. But there are some drawbacks of this mechanism:
1. There is only 30-day buffer, there might be longer time required for a client, e.g. transferred by sea, or a replacement required due to data damage;
2. It's hard for the client to do on-chain data verification;
3. Once there is a wrong-proving (proveCommit fails), which happens sometimes, resealing takes time and data re-tranferring is required. 

And, in the current mechanism, we already separate the power onboarding from proveCommit to the first WindowedPoSt. This proposal is make it more explicit.

## Design

The problem is that every proveCommitted sector is put into a partition of a particular deadline, which will be checked periodically if WindowedPoSt is done in a particular timeframe. All sectors will be scanned every 24 hours.

This proposal is to separate the proveCommit and deadline assignment of a sector into different methods, so that the sealing service provider could generate a proof of a sector and put it on chain, but not assign a deadline for it, after the data is transferred to the SP, the SP can invoke another method to add the sector into a deadline.

This proposal allows the sealing provider submit ProveCommit first, and the client can verify data using onchain information before assign proveCommitted sectors into deadlines. 

## Implementastion
_To be discussed_

In the initial thinking, there are three new methods to be added: 
- ProveCommitSector2: similar as ProveCommitSector, but not assign the sector to a deadline 
- ProveCommitAggregate2: similar as ProveCommitAggregate, but not assign sectors to deadlines 
- OnBoardSectors: Assign a bunch of proveCommited sectors to deadlines


## Discussion

This should be no problem for CC sectors to arbitrary buffer between the PoRep and first PoSt, however, for a sector having deals, as the current mechanism, the sector should be onboard before the earliest StartEpoch of all the deals in the sector, otherwise, it fails.

The question is that whether we should support the StartEpoch change in an unactivated deal? It will definitely help SaaS if this change is allowed. This requirement is for further discussion, out of this proposal's scope. 


## Consideration

The implementation of this proposal will be simply add two more methods to the miner actor (?), therefore no impact to the current system. In other words, it is fully back-compatible.

Comments and suggestions are welcomed!
