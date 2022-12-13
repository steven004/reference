Author: Venus Team

## Simple Summary

Decoupling proveCommit and deadline assignment of a sealed sector.

## Abstract

Given the limitations on current state of network infrastructure across the globe,  in order to achieve any meaningful Sealing-as-a-Service in scale, we need to leave enough time to move sealed sector from a SaaS provider to SPs through either online transferring or offline transport. In this discussion, we propose new sector proving methods that separate sector proveCommit and on-chain sector power activation to allow longer buffer time for data movement between SaaS provider and SPs.

## Motivation

Sealing-as-a-Service has grown from [an idea](https://filecoinproject.slack.com/archives/C0320QCV8SY/p1644244480718559) summarized in [a Google doc](https://docs.google.com/document/d/1zF1F6Q_Ya6IOUe9nfDOa0bNifvIa0TY4nR89omxy0kQ/edit) to one of the priorities among Filecoin data-onboarding roadmap. There are many potential merits of SaaS which includes lowering entry to Filecoin network, further separation of roles, and the possibility of creating [new line of business](https://github.com/filecoin-project/lotus/discussions/9079).

As both Lotus and Venus team are working on [SaaS APIs designs](https://github.com/filecoin-project/lotus/discussions/9079) and make adjustments to support SaaS, there is one thing left in the puzzle to be solved before SaaS business can be done in scale -  the time window is too short for a on-chain proved sector to be moved from one place to another. For any active sector, the SP has to do WindowedPoSt within 24 hours or less, or it will be declared as faulty. It is evident that 24 hour is not practical for sectors to be moved from a SaaS provider to an SP.

In the current protocol design, we have already separated proveCommit from the first WindowedPoSt during storage power onboarding. This proposal tries to make it more explicit.

## Design Rationale

The problem is that every proveCommitted sector is put into a partition of a particular deadline, which will be checked periodically if WindowedPoSt is done in a particular timeframe. All sectors will be scanned every 24 hours.

There are two design we see that could realize the goal of this proposal.

### Design 1

Taking advantage of the 30-day maximum waiting period between the preCommit and proveCommit, a SaaS provider could withhold proveCommit message and only send the message until the data is transferred to its client (SP) successfully. But there are some drawbacks of this mechanism:

1. 30-day buffer might not be enough time for a SaaS provider to transport sectors to a client (SP), e.g. transferred by sea, or a replacement required due to data damage;
2. No way for the client to do on-chain data verification;
3. If any error occurred during sealing (proveCommit fails), which happens sometimes, resealing takes time and data re-tranferring is required. 

### Design 2

Second design is to separate the proveCommit and deadline assignment of a sector into different methods, so that the SaaS provider could generate proofs of a sector and put it on chain without assigning a deadline to it. Once the sectors are transferred to the SP, the SP can then invoke another method to add the sector into a deadline.

This design allows the SaaS provider to submit ProveCommit first, and the client can then verify data using onchain information before assign proveCommitted sectors into deadlines. We think that this might be preferred design as it is less destructive to current protocol implementation and have great backwards compatibility. 

## Implementastion
_To be discussed_

In the initial thinking, there are three new methods to be added (following design 2): 
- ProveCommitSector2: similar as ProveCommitSector, but no assigning the sector to a deadline 
- ProveCommitAggregate2: similar as ProveCommitAggregate, but no assigning sectors to deadlines 
- OnBoardSectors: Assign a batch of proveCommited sectors to deadlines


## Discussion

This should be no problem for CC sectors to arbitrary buffer between the PoRep and first PoSt, however, for a sector having deals, as in the current mechanism, the sector should be onboarded before the earliest StartEpoch of all the deals in the sector, otherwise, it fails.

The question is that whether we should support the StartEpoch change in an unactivated deal? It will definitely help SaaS if this change is allowed. This requirement is for further discussion and is out of scope of this proposal. 


## Consideration

The implementation of this proposal will be simply add two more methods to the miner actor (?), therefore no impact to the current system. In other words, it is fully back-compatible.

Comments and suggestions are welcomed!
