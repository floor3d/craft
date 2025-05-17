
# RAFTimus

## RAFT Basics

- x separate servers, each being in one of 3 states
  - Leader 
    - Only one should exist in normal execution
  - Follower
    - Every other server; just responds to stuff from candidate(s) / leader
  - Candidate
    - State when leader fuckin dies

- Divided into **terms** (int)
  - Starts with **election**; if no leader is elected, a new term starts
  - If leader dies, a new term starts
  - Each server must store term int
  - Term # exchanged with all comms; if a diff term is higher, the server updates its term to that

- RPCs
  - Only two are necessary
  - **RequestVote**: initiated by candidates during elections
  - **AppendEntries**: initiated by leaders, both a heartbeat and to replicate log entries
  - Retries in sending these if no response within heuristic timeframe

## Leader Election

- Start state = Follower
- Grabs heartbeats from Leader (AppendEntries w/ no logs)
- **Election Timeout**: after *ElectionTimeout* milliseconds with no *AppendEntries*, follower is like, aint no leader no more. time for new eleckshin
- How to create new election
  - Increment term int
  - Transition to *candidate* state
  - Vote for itself
  - Send *RequestVote*s to other servers
    - Three possibilities: it wins, another leader wins, or nothing happens for X heuristic milliseconds

- if: Wins
  - Gets majority of votes, so transition to *leader* state and send a heartbeat message out to establish authority

- if: Loses
  - Still trying to get votes, but it receives *AppendEntries*; check that term # >= my current term, then if so, become follower
  - If term # < my current term, ignore RPC and stay a candidate

- if: Nothing happens
  - No majority is achieved
  - Randomize election timeout: choose from fixed interval, i.e. 150-300ms
  - Restart election timeout @ start of election; wait for *ElectionTimeout* milliseconds before starting the next election

## Log replication

- Only leaders service client requests
- Leader gets client request, adds it to its log, and sends *AppendEntries* **in parallel**
  - If this fails i.e. network is cooked or followers are slow, Leader re-sends *AppendEntries* as long as it takes
- ??? zone begin
- Logs hold the state machine command and the current term int
- Once the leader has replicated the entry on the majority of servers, this is *committed*. 
- Leader keeps track of highest index it knows to be committed, & **includes it in future AppendEntries** so that the other servers find out
- If a follower learns that a log entry is committed, it applies the entry to its local state machine (in log order)
- Leader handles inconsistencies in logs of its followers by replacing it with its own
  - Leader must find latest log entry where the two logs agree, then delete everything after that in the follower, then send all the new stuff to that follower
- 
