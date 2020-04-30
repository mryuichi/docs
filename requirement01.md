## Title: <title>

### Submitter(s): 

Ryuichi Matsukura

### Reviewer(s):

<reviewers>

### Tracker Issue ID:

<please leave blank>

### Use case reference(s):

https://github.com/w3c/wot-architecture/blob/master/USE-CASES/smart-agriculture.md

### Affected WoT Deliverables:

Architecture
Thing Description
Profile

### Requirements:

- device virtualization<br>
Enable to respond to the requests from consumers instead of the orignal correspoding to the virtual device and beeing offline<br>
Create and keep TD for a virtual device that has the same interaction affordances as those of the original thing, but the different URL that points to the intermidiary

- States for things (devices)<br>
Enable to maintain states of things in TD<br>
  for examples, such states should be managed:<br>
  - running: thing is available (has valid URL) and able to reply to consumers's requests<br.
  - sleeping: thing is available (has valid URL) but able to reply to consumers's requests becuase of it(device) suspending<br>
  - stopping: thing is disabled (has no URL)
  
Comments: may separete discussions for states during onbording process to states after onboarding. Perhaps application developpers have much interested in states after onborading.

- units
Enabel to refer the definitions for unit specifined in a various standards

### Related standards:

<list related standards>

### Other references:

<additional references that provide more context>

### Comments:

<additional comments>
