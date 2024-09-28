# Time Locks


* Locktime: (absolute locktimes)  
  * 0 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;No locktime
  * < 500 billion &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Block DAA score <br/>
  * \>= 500 billion &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Unix timestamp (milliseconds) <br/>

* Sequence: (relative locktimes)
    - bit 63: 
       - 0 - sequence lock enabled
       - 1 - sequence lock disabled
    - bits 0-31:
        actual relative locktime, unsigned int32
        
 * Disable relative time-based timelocks (Ignoring the type flag)
 * Adds special functions to support timelocks when building a script: AddLockTimeNumber, AddSequenceNumber.

---

Docs: [https://github.com/kaspanet/docs/blob/main/Reference/Timelocks.md](https://github.com/kaspanet/docs/blob/main/Reference/Timelocks.md)
