In this crosschain swap architecture, a centralized relayer service orchestrates the entire swap process between
users and resolvers. When a user initiates a swap, they first approve the relayer contract to spend their
source tokens, then submit their order to the relayer service. The relayer broadcasts this order to all
registered resolvers along with the current market price. Resolvers monitor these broadcasts and when one finds
the price acceptable, they commit to fulfilling the order through the relayer's API, which starts a 5-minute
execution timer. The committed resolver then deploys escrow contracts on both source and destination chains with
safety deposits, and notifies the relayer once ready. The relayer transfers the user's pre-approved funds from
the user to the source escrow, while the resolver deposits their own funds into the destination escrow. Upon
completion, the resolver notifies the relayer, who then reveals the secret on the destination chain to unlock
funds for the user and return the safety deposit to the resolver. The resolver uses this same secret to withdraw
the swapped funds and their safety deposit from the source chain. If the resolver fails to complete within the
5-minute window, the order becomes available for any other resolver to rescue - they can complete the trade and
claim the original resolver's safety deposits as a penalty reward. This design minimizes resolver capital
requirements (only safety deposits needed), prevents gas wars through single-resolver commitment, ensures order
completion through the timeout rescue mechanism, and provides coordinated execution through the relayer's
API-driven orchestration.
