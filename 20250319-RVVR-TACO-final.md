# TACO Dao Audit - For TACO DAO

## Security Assessment

March 19th, 2025 - Final Report

Prepared By:
RIVVIR Tech, LLC, Austin Fatheree

### About RIVVIR Tech, LLC

RIVVIR Tech LLC is a Texas based company that provides technology consulting and engineering services.  Among other services, we perform security audits of MOTOKO based canisters and have an extensive history of building MOTOKO based canisters, participating on both sides of the security audit table, and have participated in ICRC Working Groups.  For information regarding our services, please reach out to austin at rivvir.com.

### Disclosure:

This security assessment was conducted within a specified timeframe and depended substantially on data furnished by the client, along with its affiliates and partners. Consequently, it's important to acknowledge that the insights presented in this report do not represent an exhaustive enumeration of all potential security vulnerabilities or anomalies within the evaluated system or codebase. This report highlights key findings as per the information available and the scope of the assessment during the conducted period. We make no warranties or guarantees about the efficacy or security of the audited contracts and these findings are provided as informational content only.  Clients must rely on their own judgment, implementation, and must make their own warranties and guarantees of their published code.

© 2025 by RIVVIR Tech, LLC

All rights reserved. RIVVIR Tech hereby asserts its right to be identified as the creator of this report in the United States, United Kingdom, EU, and Globally.

This report is considered by RIVVIR Tech to be business confidential information; it is licensed to the TACO DAO for informational purposes. Material within this report may not be reproduced or distributed in part or in whole without the express written permission of RIVVIR Tech LLC.

## Executive Summary

### Engagement Overview

TACO DAO engaged RIVVIR Tech LLC to review its DAO, Spam, and Neuron snapshot infrastructure.  From February 11th to March 7th Austin Fatheree reviewed the contract, made suggestions, investigated the contract and produced a draft. Over the next weeks, TACO DAO refactored codes based on our recommendation and the recommendation of other auditors.  On March 19th we evaluated the response of TACO DAO to the original draft and updated our report.

### Project Scope

Our efforts were focused on identifying uneeded complexity, security issues, poor code quality, potential exploits, and other issues involved in the backend canister controlled by the TACO DAO that performs various DeFi and DAO operations.

### Summary of Findings

The audit uncovered a few issues that we felt need to be addressed and/or publicly justified by the team before moving forward with the deployment of the DeFi Vectors canister contract. The TACO DAO team has responded to these with either fixes or justifications of their choices. Only one moderate issue remains that may affect User Access, but this likely only affects users deliberately trying to create the instance and is limited in damage to only that user.

The code is well structured and well documented and we find no high security vulnerabilities remain in the current version.

Exposure Analysis

| Severity     | Count |
|--------------|-------|
| High         |      0|
| Moderate       |      1|
| Low          |      0|
| Informational|      0|

Category Breakdown

| Category             | Count |
|----------------------|-------|
| User Access      |      1| 


#### Notable Findings

Upon revision by TACO DAO we have not identified any High Severity open items.  The remaining item either has plans in place to address, reporting, or are highly difficult issues to take advantage of. 

## Project Goals

Our goals in this project is to make sure that the TACO DAO canisters do what they are intended to do. We sought to:

- check motoko coding conventions and best practices
- address intercanister workflows
- identify exploits that would allow the stalling of the canister's workflows.
- identify potential cycle drain and DoS attacks.
- make sure upgrades work appropriately

## Project Targets

The initial Draft was performed against the commit c4bc1cc0db2a0fb4c62cdb71c73f4db8825a1afc at https://github.com/wilaq/DAO-2.0. The final draft was completed against 586f66365e5bdb1ca001c38c5597137f4f04218c.

We audited the backend motoko files.  An audit of the web application was not done.

## Code Maturity

| Category                               | Summary| Result           |
|----------------------------------------|------------------------------------------------|------------------|
| Arithmetic                             | Mature. While the use of floating-point arithmetic introduces a risk of rounding errors, the project could benefit from a transition to fixed-point arithmetic to mitigate this risk.| Strong           |
| Auditing and Logging                               | The current implementation focuses on error logging with some logging for positive events. Implementing comprehensive logging for key state transitions, alongside error logging, is recommended to improve auditability and transparency.| Moderate         |
| Authentication / Access Controls       | Mature. The smart contract employs Principal-based access controls for critical functions, effectively managing access and permissions based on the caller's identity. In addition significant thought has been put into limiting ingress from outside the IC and providing spam controls| Strong         |
| Complexity Management                 | The code has been simplified across a number of dimensions. Some large logic functions remain that could be refactored. | Moderate           |
| Cryptography and Key Management        | Mature. The use of the Internet Computer's Principal and identity management infrastructure provides a solid foundation for secure interactions with the contract. However, the code review did not specifically target cryptographic operations, as these are largely abstracted by the platform.| Strong           |
| Data Handling                          | Mature. Sufficient data checks, storage, and data handling.             | Strong           |
| Documentation                          | Mature. The smart contract and associated types are well-documented within the codebase. | Strong           |
| Maintenance                            | Underdeveloped. The lack of a clear migration strategy and the presence of unused imports and references indicate potential challenges in maintaining and upgrading the contract. Implementing a migration framework and clean-up of unused code can improve long-term maintainability.| Moderate         |
| Memory Safety and Error Handling       | Errors are handled in both sync and async scenarios and memory overflow are handled.| Strong           |
| Testing and Verification               | Extensive integration tests were provided.                                                                                                                                | Moderate             |

## Identified Issues

### Issue: RVVR-TACOCDAO-004 - Error Feedback in DAO Canister for Maximum Updates Exceeded

**Type:** User Access
**Severity:** Moderate
**Difficulty:** Medium

This section addresses error conditions when the maximum updates in the DAO canister are exceeded. Refer to the code [here](https://github.com/wilaq/DAO-2.0/blob/706c02737a53dab2e78c40c73611e376006fbaba/src/DAO_backend/DAO.mo#L452) to understand the error handling implementation.

If a user reaches this number of updates, what is there recourse?  Dissolving neurons takes a while and users may be left without access to their neuron votes until they get the right number to dissolve.

#### TACO DAO Response

This is an edge case as long as the limit isn/t set too low.  We agree the code should be updated but we will wait until a future update since this would only affect potentially nefarious users.

#### Response evaluation

The issue remains in place, but it is a user footgun and does not provide a security issue for anyone but the user creating an extreme number of neurons.


## Identified Issues that were addressed by TACO DAO.

The following issues were identified during our initial review of the canister code and are provided for informational purposes only as they may be instructive to other projects building on the Internet Computer.  TACO DAO has addressed the threat of any security issues from the below listed items.

### Issue: RVVR-TACONUR-001 – Storage Limitation for Neuron Snapshot Data

**Type:** Informational / Scalability  
**Severity:** Low  
**Difficulty:** Medium

**Description:**  
The neuron snapshot module in the TACO DAO system is designed to capture and aggregate the voting power from participating neurons. However, the current implementation is constrained by a 2MB storage cap, which restricts the module’s ability to handle more than approximately 10,000 neurons in a single snapshot request. This limitation becomes significant as the DAO scales, potentially leading to failures in the snapshot query process by first and third party dependent systems when the number of neurons exceeds the storage threshold.

**Recommendation:**  
Other mechanisms exist that provide a paging mechanism to divide the neuron snapshot data into manageable chunks. We recommend removing the non-paginated queries so that no one becomes dependent on them.

**Code Reference:**  
The relevant code section can be reviewed [here](https://github.com/wilaq/DAO-2.0/blob/482b9022757138618823bd5461cef18e3ac875a7/src/neuron_snapshot/neuronSnapshot.mo#L149).

#### TACO DAO Response

The issue links to get_neuron_snapshot_info.

Of which the return type is a record that does not hold any arrays. So pagination is not needed. I think that function got confused with getNeuronDataForDAO which sends large chunks of data and allows pagination.

#### Response evaluation

Indeed this type return on this function hols just three items and thus can produce much more data. While still not unlimited in nature we do not expect this to affect performance for anything but the very largest DAOs with an abnormal number of neurons.

### Issue: RVVR-TACONUR-002 – Inefficiency in Neuron Snapshot Data Processing

**Type:** Scalability  
**Severity:** Low  
**Difficulty:** Medium

**Description:**  
The existing implementation iterates over the complete collection of neurons, applying a mapping function to transform each entry, even though subsequent operations only require a subset of these entries. This unnecessary processing amplifies computational overhead, particularly when the neuron population is large, leading to suboptimal performance and increased cycle usage. Queries are currently "free," but they may not always be and queries from other canisters are upgraded to update calls.

**Impact:**  
For production deployment, this inefficiency may cause slower response times and higher resource consumption. In worst-case scenarios, the approach can contribute to performance bottlenecks, thereby affecting the overall responsiveness and scalability of the DAO's neuron snapshot operations.

**Recommendation:**  
It is advisable to defer the mapping operation until after paging has been applied. By first dropping the unnecessary front segment of the data and taking only the required subset, the system can then apply the mapping, resulting in reduced processing time and more efficient resource usage. Such a change will mitigate the risk of costly processing cycles and improve overall system performance.

**Example Improvement:**

```motoko
// Instead of mapping the entire list and then slicing,
// apply drop/take first and map only the needed subset:
let pagedNeurons = fullNeuronList.drop(startIndex).take(pageSize);
let processedNeurons = Vector.map(pagedNeurons, func(neuron) : ProcessedNeuron {
  // mapping logic here
});
```

Implementing this change will ensure that only the necessary neurons are processed, thereby optimizing both performance and resource efficiency during snapshot operations.

#### TACO DAO Response

Paging is not ended here as its max 100bytes. However the changes have been processed, small optimisation in getNeuronDataForDAO: 77691e8ac60e20e0b32947c1f8a5979fca3aecca

#### Response evaluation

The new code relieves the scalability concern.

### Issue: RVVR-TACONUR-003 – Unnecessary Parameter Retrieval in add_neuron_snapshot Function

**Type:** Optimization / Efficiency  
**Severity:** Low  
**Difficulty:** Medium

**Description:**  
In the add_neuron_snapshot function, the code retrieves parameters every time the function returns true. This retrieval may be unnecessary if these parameters are not ultimately required for further computations. The GitHub [code reference](https://github.com/wilaq/DAO-2.0/blob/482b9022757138618823bd5461cef18e3ac875a7/src/neuron_snapshot/neuronSnapshot.mo#L346-L347) shows a snippet where parameters are obtained even though only a boolean check is needed to execute the function logic.

**Exploit/Impact Scenario:**  
While not representing a security vulnerability, this inefficiency can lead to suboptimal performance, particularly in high-frequency calls or in scenarios where resource usage must be tightly managed. The repeated retrieval of parameters can add unnecessary overhead, slowing down processing cycles in the neuron snapshot operations.

**Recommendation:**  
It is advisable to conditionally retrieve parameters only in cases when the function outcome is true, rather than retrieving them on every invocation. One potential optimization is to delay the parameter retrieval until it is verified that they are indeed needed. This can be achieved by restructuring the code with an early return check. For instance:

**Code Excerpt (Current):**

```motoko
if (/* condition that determines true outcome */) {
  // Unconditionally retrieve parameters
  let params = getParams();
  // use params in add_neuron_snapshot
  return true;
} else {
  return false;
}
```

**Optimized Code Example:**

```motoko
if (/* condition that determines true outcome */) {
  // Only after confirming outcome, then retrieve parameters
  let params = getParams();
  // Further processing with params
  return true;
} else {
  return false;
}
```

By restructuring the code in this way, the function avoids the performance penalty associated with unnecessary parameter retrieval calls.

**GitHub Reference:**  
[DAO-2.0/neuronSnapshot.mo#L346-L347](https://github.com/wilaq/DAO-2.0/blob/482b9022757138618823bd5461cef18e3ac875a7/src/neuron_snapshot/neuronSnapshot.mo#L346-L347)

#### TACO DAO Response

By only retrieving the parameters right before returning true, which means it won't happen in cases where the function returns false earlier.

#### Response evaluation

The new code relieves the issue with unneeded retrieval.

### Issue: RVVR-TACONUR-004 – Inefficiency of Array Append Operations in Neuron Snapshot Module

**Type:** Optimization / Efficiency  
**Severity:** Low  
**Difficulty:** Medium

**Description:**  
The neuron snapshot module in our DAO code incorporates an array append operation (as seen in the code snippet at GitHub [L456-L457](https://github.com/wilaq/DAO-2.0/blob/482b9022757138618823bd5461cef18e3ac875a7/src/neuron_snapshot/neuronSnapshot.mo#L456-L457)). Although using arrays is a straightforward approach for accumulating data, the append operation on arrays in Motoko can lead to inefficiencies when processing large neuron datasets.

**Potential Inefficiencies:**  
1. **Memory Reallocation:** Array append operations often require reallocation, which can introduce significant overhead in terms of CPU cycles and memory usage when handling extensive datasets.
2. **Cycle Consumption:** Repeated allocations and copying of data cause inefficient cycle usage—particularly critical in Internet Computer canisters, where cycles are a valuable resource.
3. **Scalability Bottlenecks:** As the system processes increasing amounts of neuron data, the linear cost of array operations may lead to slower response times and degraded performance.

**Recommendation:**  
- **Switch to Vectors:** Vectors are designed for dynamic data and efficiently handle costly operations like appending. They reduce the frequency of reallocations by growing in capacity, which improves cycle efficiency.

#### TACO DAO Response

Eliminated Array.append by switching to Vector


#### Response evaluation

The new code relieves the scalability issue.

### Issue: RVVR-TACONUR-005 – Controller Check and Argument Size Validation in SNS Canister

**Type:** Security / Validation  
**Severity:** High  
**Difficulty:** Low

**Description:**  
The code segment in the neuron snapshot canister (see GitHub link below) includes a controller check that ensures only authorized principals invoke SNS canister functions. The check does not utilize an explicit app canister variable. It is our understanding that only the SNS governance canister can be the controller of a DAO App Canister.  As a result, the Treasury, Swap, and other DAO canisters would not be able to call these functions. Only the SNS governance canister would be able to call it and it would provide no functionality.

Additionally, the audited code lacks argument size validation, particularly for Nat variables received as inputs. Without upper bound checks, there exists the potential for malicious actors to submit oversized values. This could lead to unintended computational overhead or cycle exhaustion. Best practices recommend that high-risk input values (such as Nat) be validated to ensure they fall within acceptable limits.

**Recommendations:**  
- Introduce an explicit app canister set alongside the existing controller check. This will allow the SNS canister to differentiate between calls from the DAO Governance and calls from the approved app canister, enabling role-specific validations.
- Implement rigorous argument size validation for Nat, Blob, and Text variables. This can be realized by using helper functions or inline conditionals that check whether a given input does not exceed a predefined maximum. For instance:

```motoko
// Helper function for Nat validation
func validateNat(input: Nat, maxAllowed: Nat) : Bool {
  return input <= maxAllowed;
}

// Controller and app canister validation
let appCanister: Principal = Principal.fromText("<APP_CANISTER_ID>");
if (caller != DAOprincipal and caller != appCanister) {
  return #err(#NotAuthorized);
}
```

**Code Reference:**  
GitHub – Neuron Snapshot Controller Check:  
https://github.com/wilaq/DAO-2.0/blob/482b9022757138618823bd5461cef18e3ac875a7/src/neuron_snapshot/neuronSnapshot.mo#L658-L659

#### TACO DAO Response

The inspect system function is not triggered during inter-canister calls, only during external calls. Since the Neuron Snapshot canister is designed to be used exclusively by the DAO canister, and not directly by external users, the current controller-only check is appropriate for our access control requirements. If future stats or logging functionality is needed, we can implement a separate canister for those specific queries. The current implementation properly limits access while minimizing cycle usage for our intended workflow.

#### Response evaluation

The response satisfies the concern.

### Issue: RVVR-TACONUR-006 - Resetting 'neuron_snapshot_importing' on Half-Finished Data During Upgrades

**Type:** Reliability / Safety
**Severity:** High
**Difficulty:** Medium

**Overview:**
During system upgrades, there is a potential risk where data import processes might be interrupted, leaving the 'neuron_snapshot_importing' variable in a partially set state. Such half-finished data can lead to inconsistent states, which may compromise the integrity of the snapshot data and impair subsequent upgrade attempts. Resetting this variable as part of the upgrade cleanup process ensures that the system does not proceed with corrupt or incomplete data, thereby mitigating risks that could lead to data corruption or unpredictable system behavior.

**Rationale:**
- **Data Integrity:** Ensures that any half-finished snapshot data does not conflict with new data or introduce inconsistencies.
- **Operational Safety:** Prevents the system from running in an undefined state that might lead to further errors or operational downtime.
- **Upgrade Robustness:** Guarantees that each upgrade cycle starts with a known, clean state for the snapshot data import, simplifying troubleshooting and reducing failure cases.

**GitHub Reference:**
[Resetting neuron_snapshot_importing Variable](https://github.com/wilaq/DAO-2.0/blob/706c02737a53dab2e78c40c73611e376006fbaba/src/neuron_snapshot/neuronSnapshot.mo#L623)

**Code Example for Safe Handling:**
The following is an example pseudocode demonstrating how to safely reset the `neuron_snapshot_importing` variable during an upgrade process:

```
// Check if the snapshot import process was incomplete
if (neuron_snapshot_importing && isUpgradeInProgress()) {
    // Log the detection of half-finished data
    log(\"Incomplete neuron snapshot import detected. Resetting state.\");
    
    // Reset the variable to ensure clean state for re-import
    neuron_snapshot_importing := false;

    // Optionally, mark snapshot as invalid or trigger a safe re-import procedure
    markSnapshotAsInvalid();
    initiateSafeSnapshotRecovery();
}

function isUpgradeInProgress() {
    // Implementation to verify if the system is currently in an upgrade state
    return getSystemState() === \"UPGRADING\";
}

function log(message: string) {
    // Log the message to an audit log system
    console.log(message);
}
```

#### TACO DAO Response

Added the system func preupgrade that resets neuron_snapshot_importing-status-timeout.

#### Response evaluation

The new code satisfies the concern.

### Issue: RVVR-TACONUR-007 – Risk of Inadvertent Snapshot Storage Due to Unchecked Await Returns

**Type:** Reliability / Safety  
**Severity:** High  
**Difficulty:** Medium

**Description:**
The neuron snapshot process in the DAO relies on asynchronous calls when interacting with the SNS subnet (e.g., for taking neuron snapshots). However, the current implementation does not re-check the status of critical control variables (e.g., CANCEL_NEURON_SNAPSHOT) immediately after each await call. If an upgrade or network disruption triggers cancellation while awaiting a response, the snapshot process may inadvertently continue to complete, thereby storing partial or incomplete snapshot data. This can compromise the integrity of snapshot data and lead to inaccuracies in voting power calculations.

**Exploitation Scenario:**
For example, during a network stall or upgrade on the SNS subnet, the snapshot process may enter an await state. If during this time the CANCEL_NEURON_SNAPSHOT flag is set to true but is not re-checked immediately after the await returns, the process may resume and continue to store a half-completed snapshot. This in turn could result in erroneous voting power data and potentially allow compromised DAO decisions.

**Recommendations:**
- **Immediate Re-Check:** Insert checks immediately after every asynchronous await call to verify if CANCEL_NEURON_SNAPSHOT (or any other control flag) has been set. If cancellation is detected, abort the process, reset snapshot state, and log the event.
- **Helper Functions:** Refactor the code to encapsulate post-await flag verifications in a helper function (e.g., `checkControlFlags()`), reducing duplication and improving clarity.
- **Robust Logging:** Enhance logging to capture cancellation events and any errors encountered during the post-await verifications. This improves auditability and aids troubleshooting.

**GitHub Reference:**
The relevant code section can be reviewed [here](https://github.com/wilaq/DAO-2.0/blob/706c02737a53dab2e78c40c73611e376006fbaba/src/neuron_snapshot/neuronSnapshot.mo#L308).

**Example Source Code Snippet:**
```
// Pseudocode Example: Ensure control flag re-check after await
if (await someAsyncCall()) {
  if (CANCEL_NEURON_SNAPSHOT) {
    Debug.print("Snapshot process cancelled during await; aborting snapshot storage.");
    reset_neuron_snapshot();
    return;
  }
  // Continue processing
}
```

#### TACO DAO Response

Rechecking CANCEL_NEURON_SNAPSHOT after every await

#### Response evaluation

The new code satisfies the concern.

### Issue: RVVR-TACONUR-008 – Deficiencies in Logging Strategy for the Neuron Snapshot Canister

**Type:** Informational / Auditing and Logging  
**Severity:** Low  
**Difficulty:** Medium

**Description:**
The current implementation of the neuron snapshot canister employs minimal logging—primarily using simple debug prints for error conditions—without a structured mechanism to capture positive operational events or state transitions. This lack of comprehensive logging hinders effective monitoring and auditability of the snapshot process, making it difficult to trace state changes during asynchronous operations or diagnose issues in production environments using the latest replica software capabilities.

**Impact/Exploit Scenario:**
Without robust logging, transient errors or subtle inconsistencies during asynchronous snapshot processes (such as after await calls or during cancellation handling) may remain undetected, impeding effective troubleshooting and post-incident analysis. This reduces operational transparency and may result in prolonged downtime when issues arise.

**Recommendation:**
1. **Enhanced Debug Logging:** Insert detailed debug statements at critical junctions, particularly immediately before and after asynchronous calls, and immediately after re-checking control flags (e.g., CANCEL_NEURON_SNAPSHOT) to capture their state.

2. **Structured Logging Framework:** Replace basic Debug.print calls with a structured logging mechanism that categorizes logs by severity (INFO, WARN, ERROR) and includes contextual metadata (e.g., current snapshot ID, operation status, and timing information). For example:

```motoko
func logStateTransition(event: Text, context: {snapshotId: Nat; status: T.NeuronSnapshotStatus}): () {
  Debug.print(event # " | Snapshot: " # Nat.toText(context.snapshotId) # " | Status: " # debug_show(context.status));
}
```


3. **Periodic State Logging:** Configure periodic logging to capture key state variables (such as the current snapshot id, number of neurons processed, and voting power aggregates) to create an audit trail for system performance metrics.

4. **Log Aggregation and Monitoring:** Ensure the logging framework is integrated with existing production log aggregation tools, facilitating real-time monitoring and effective troubleshooting.

**Summary:**
Enhancing the logging framework in the neuron snapshot canister is essential to improve system observability and auditability. Structured, comprehensive logging will help in diagnosing issues quickly, maintaining consistent monitoring of critical operations, and ensuring operational transparency as the system scales.

#### TACO DAO Response

Added a logging module in both DAO and SNS snapshot canister:
(see following commit: ad54b3260234765622610f156e11aa901a03ec13)

#### Response evaluation

The new code satisfies the concern.


### Issue: RVVR-TACONUR-009 – Permission Types for TacoDao Neurons

**Type:** Informational / Access Control
**Severity:** High
**Difficulty:** Low

**Description:**

This section evaluates the snippet concerning permission types in TacoDao neurons. The code enforces that only specific permission combinations—namely `[4, 3]` or `[3, 4]`—are considered valid, as evidenced by the condition:

```
if (permission.permission_type == [4, 3] or permission.permission_type == [3, 4]) {
  Vector.add(tacoPrincipal, p);
};
```

The design intent is to limit TacoDao neurons to a strict set of permission configurations typical of a 'hot key'. By explicitly checking for these two combinations, the implementation ensures that any permission list differing from `[4, 3]` or `[3, 4]` is silently ignored. 

**Exploit/Impact Scenario:**

Should an attempt be made to assign any permissions outside of the accepted pairs, the operation will not add the principal to the `tacoPrincipal` vector. This may cause confusion or silently leave people out of the DAO.

**Recommendation:**

Update the code to check for 3 and 4, but not exclusively. Other permissions need not invalidate the ability to make allocations.

**Code Reference:**

For further context, review the code snippet found [here](https://github.com/wilaq/DAO-2.0/blob/706c02737a53dab2e78c40c73611e376006fbaba/src/neuron_snapshot/neuronSnapshot.mo#L420).

#### TACO DAO Response

Added snapshotTimerId and canceling each time before creating new timer.

#### Response evaluation

The new code satisfies the concern.

### Issue: RVVR-TACONUR-010 – Race Condition due to Untracked Timer

**Type:** Reliability / Safety
**Severity:** Low
**Difficulty:** Low

**Description:**
In the neuron snapshot canister, operations rely on asynchronous timers to trigger periodic snapshot updates. The implementation currently uses Timer.setTimer following asynchronous calls. Without proper tracking or cancellation, these timers can overlap, causing race conditions during snapshot processing. Network delays or suspended operations exacerbate this issue, potentially allowing multiple timers to initiate concurrently.

**Exploit/Impact Scenario:**
Multiple active timers can lead to overlapping snapshot operations. This may result in inconsistent state updates, race conditions, partial data collection, or even excessive cycle consumption if the canister becomes overloaded with redundant tasks.

**Recommendation:**
Implement a timer management strategy:
1. Store the timer identifier in a stable variable.
2. Before scheduling a new timer using Timer.setTimer, check if a timer is already active and if so, cancel it using Timer.cancelTimer.
3. Perform safeguard checks after asynchronous operations to verify system state before launching additional timers.

**Code Reference and Example:**

Consider the current usage:

```
ignore Timer.setTimer<system>(#seconds 2, func() : async () { await* take_neuron_snapshot_tick() });
```

This approach can lead to multiple timers being active simultaneously. Instead, use the following improved pattern:

```
if (snapshotTimerId != 0) {
    Timer.cancelTimer(snapshotTimerId);
}
snapshotTimerId := Timer.setTimer<system>(#nanoseconds(SNAPSHOT_INTERVAL), updateSnapshot);
```

#### TACO DAO Response

Added snapshotTimerId and canceling each time before creating new timer.

#### Response evaluation

The new code satisfies the concern.

### Issue: RVVR-TACOCDAO-001 - DAO Canister Token Update Mechanism

**Type:** DAO Functionality / Token Management
**Severity:** High
**Difficulty:** Low

**Description:**
The DAO canister’s token update function must handle updates to token data as governed by external changes (e.g. adjustments in fee parameters) while preserving immutable historical metadata. In the current implementation, when a token is added or updated, a new entry is written to the `tokenDetailsMap`. The code snippet includes a conditional branch where if the token exits, it's metadata is not updated. This could lead to difficulty in updating active token metadata when a remote token decides to change their Fees(See $BOB).

**Recommendation:**
Update the token metadata even when active tokens are found as the metadata may have changed. If you detect a change, you may want to auto pause the token until the DAO can confirm that they want to proceed under the new values..

**Code Reference:**
Review the following snippet from the DAO code where the token update occurs:

```
case (?details) {
  if (details.Active) { return #ok("Token already exists") } else {
    Map.set(tokenDetailsMap, phash, token, { metadata with Active = true; isPaused = false; epochAdded = Time.now(); balance = 0; priceInICP = 0; priceInUSD = 0.0; pastPrices = [] });
  };
};
```

#### TACO DAO Response

Edited it for a metadata call to not go to waste and not include redundant code, metadata does get updated using the updateTokenMetadata in treasury of which the DAO timely syncs with. 

#### Response evaluation

The new code satisfies the concern.

### Issue: RVVR-TACOCDAO-002 
– Error Handling and Retry Mechanism for Treasury Token Detail Synchronization

**Type:** Reliability / Safety
**Severity:** Moderate
**Difficulty:** Medium

**Description:**

The treasury token detail synchronization process in the DAO backend relies on an asynchronous call to update token details from the treasury canister. In the existing implementation, the synchronization call is encapsulated within a try/catch block:

```
try {
  ignore await treasury.syncTokenDetailsFromDAO(Iter.toArray(Map.entries(tokenDetailsMap)));
} catch (_) {};
```

If this operation fails, the treasury may potentially continue trading based on outdated or incorrect token details. An absent or overly simplistic error handling mechanism does not offer visibility into synchronization failures nor does it attempt a recovery, which may lead to further processing with stale data and unexpected behavior.

**Exploit/Impact Scenario:**

If the synchronization call encounters an exception—due to network timeouts, canister unavailability, or transient errors—the failure is silently ignored. This could allow the treasury to operate with token details that do not reflect the current state of the ledger, possibly resulting in incorrect trading decisions and financial losses.

**Recommendation:**

1. **Enhanced Error Logging:** Instead of silencing errors, log detailed error messages to provide insight into the nature of the synchronization failure. For example, include error codes or messages from the treasury call to facilitate diagnostics.

2. **Retry Mechanism:** Implement a structured retry strategy wherein the synchronization call is reattempted a specified number of times with appropriate back-off intervals. This reduces the likelihood of transient errors causing persistent issues.

3. **Fallback Procedures:** In cases where retries continuously fail, trigger a fallback procedure. This might involve flagging the token for manual review or temporarily disabling trading for the affected asset until a successful synchronization can be assured.

4. **Notification and Alerting:** Integrate an alert system to inform administrators when synchronization failures occur after multiple retries. This ensures timely intervention.

#### TACO DAO Response

At the moment it does get synced with treasury already with every X ns (default: 15 mins). And pausing the token because other canisters can/t update won/t do any good as pausing a token on treasury is almost the same call logic. So I added logic in treasury that keeps up with when the last token update was, if its a long time back then it will have to be paused. The same does not need to happen in DAO as the treasury data sync is just for the FE.

#### Response evaluation

The new code satisfies the concern with the DAO canister and we will re-evaluate when the Treasury canister is reviewed.

### Issue: RVVR-TACOCDAO-003 -  ICRC-3 Auto-Archiving

**Type:** Informational
**Severity:** Informational
**Difficulty:** Low

**Description:**
The ICRC-3 auto-archiving mechanism is designed to efficiently manage historical data storage in the DAO canister. Instead of retaining all operational data in primary storage, the system periodically archives events and historical records. This auto-archiving provides several benefits:

1. **Reduced Data Footprint:** By archiving older records, the live canister stores only the most recent and relevant data. This ensures that the data storage remains lean and minimizes memory usage over time.

2. **Cycle Efficiency:** Archiving reduces the active data that the canister must process in real time. This lowers the computational overhead and cycle consumption during routine operations, allowing for more efficient use of cycles.

3. **Maintained History for Auditing:** Despite reducing the load on active storage, the auto-archiving process preserves the complete historical record. Archived data remains accessible for audits and forensic analysis without interfering with ongoing operations.

**Exploit/Impact Scenario:**
Under normal conditions, the DAO canister processes transactions and state updates while periodically archiving old events. In high-activity scenarios, maintaining a reduced active dataset prevents overconsumption of cycles and enhances performance. The preserved archive ensures that historical data is available for compliance and security reviews.

**Recommendation:**
Consider using ICRC-3 to emit events that you want to use off-chain and in applications instead of having to keep track of sizes and run archive routines. The ICRC-3 auto-archiving mechanism will simplify collection and reduce data on your main canisters.

**Note:**
This is only a recommendation. The current implementation keeps only a small amount of data 'live' on the canister and is robust.

For additional context, see [here](https://github.com/wilaq/DAO-2.0/blob/706c02737a53dab2e78c40c73611e376006fbaba/src/DAO_backend/DAO.mo#L384).

#### TACO DAO Response

We would rather keep the  inter canister calls to a minimum and trim at a adequate amount (past allocations default is 100 (changed from 50 after this issue), with min of 20 and max of 500)

#### Response evaluation

The was a recommendation and the chosen pathway provides no security risk.

### Issue: RVVR-TACOCDAO-005 – Informational Disclaimer: Price Assumptions and Potential Handbreaks

**Type**: Informational – Disclaimer
**Severity**: Low
**Difficulty**: Low

#### Description:
The current design of the DAO’s token update mechanism assumes that the supplied price information from external sources is accurate and reliable. Specifically, the code responsible for updating token details (e.g., in the DAO canister’s token update functionality) operates under the premise that price inputs—typically retrieved from the treasury or related sources—are “good.” This assumption is embedded in the design and in the error feedback mechanisms, with few safeguards to proactively address situations where price manipulation or unusual market events occur.

#### Exploit/Impact Scenario:
In events where an attacker or market anomaly causes a drastic fluctuation in a token's price, there is a risk that the system might fail to trigger corrective measures (or “handbreaks”) immediately. Under such circumstances, a single token could accumulate excessive allocation or, conversely, be divested abruptly. Both scenarios can lead to unintended and potentially extreme outcomes in governance and treasury operations. Although the system currently provides a basic assumption that prices are “good,” these conditions underscore the need for precautionary enhancements.

#### Recommendations:
It is recommended that future modules incorporate additional safeguards, such as dynamic autopause bands. These mechanisms would automatically restrict operations (or “pause” transactions) if price data deviate beyond predefined thresholds. By integrating such a strategy, the system could mitigate potential abuses, ensuring that drastic, unanticipated price manipulations are caught and addressed before they can destabilize the DAO. In upcoming releases, developers should consider:
• Establishing upper and lower bounds for acceptable price fluctuations.
• Implementing rate limits or automated pausing of token updates when prices breach these parameters.
• Including clear logging and internal alerts to inform operators about potential anomalies.

#### TACO DAO Response

The price information is not really used within the DAO context and is only there to provide the FE with a single query to get a lot of data. So nothing can go wrong within the canisters perse. Within treasury and the mint vault this price data is used and those have a lot of safety features.

#### Response evaluation

This data in the DAO canister does not result in a security risk, but may still cause user issue on the front end(outside the scope of this audit) and we will re-evaluate when the Treasury canister is reviewed.

### Issue: RVVR-TACODAO-006 – Size Restrictions on Function Arguments & Security Implications

**Type:** DAO Functionality / Input Validation  
**Severity:** Moderate
**Difficulty:** Medium

**Description:**
The DAO canister imposes a hard limit of 512 Bytes on function arguments. While this limit is intended to contain the risk of resource abuse and ensure predictable cycle consumption, it introduces several potential issues:

1. **Input Truncation and Data Loss:**
   - Restricting function inputs to 512 Bytes may inadvertently reject larger payloads that were expected by the caller, potentially leading to incomplete data processing or unexpected side effects.

**Recommendations:**
- **Strict Boundary Checks:** Implement explicit checks for each function that is more tailored to that function's use case.
- **Layered Validation:** Complement the size check with semantic validations. This multi-tier approach should validate both the structure and content of inputs.
- **Regular Testing:** Employ fuzz testing and boundary analysis to ensure that the truncation or rejection logic does not lead to inadvertent errors or provide loopholes for spoofing attacks.

**GitHub Reference:**
Refer to the implementation details at: [DAO_Backend/DAO.mo#L1545](https://github.com/wilaq/DAO-2.0/blob/706c02737a53dab2e78c40c73611e376006fbaba/src/DAO_backend/DAO.mo#L1545)

#### TACO DAO Response

Increased to 5000 as indeed this could make updateAllocations fail if there are a lot of tokens to be voted for. More is not needed as other calls that require arguments are expected and designed to only be made by canisters.

#### Response evaluation

This update reduces the immediate limitation.
