### **Day 54: Functional Coverage – Practical Example**

---

### **Objective:**

Apply what we learned about functional coverage to a **realistic verification scenario**. This example demonstrates how to:

* Define and use covergroups in a meaningful way.
* Collect useful metrics on test quality.
* Cross-check combinations of values.

---

## ✅ 1. Scenario: Bus Transaction Coverage

We’ll verify transactions on a simple bus interface. Each transaction has:

* `addr` (address)
* `data` (payload)
* `rw` (read/write)
* We want to ensure:

  * All address ranges are hit.
  * Both read and write ops are exercised.
  * Combinations of addresses and operation types are covered.

---

## ✅ 2. Define the Transaction Class with Coverage

```systemverilog
class BusTransaction;

    rand bit [3:0] addr;    // 4-bit address
    rand bit rw;            // 0: read, 1: write
    rand bit [7:0] data;    // 8-bit data payload

    // Coverage group to track combinations and data value bins
    covergroup trans_cov @(posedge clk);

        // Coverpoint on address range
        coverpoint addr {
            bins addr_low[4]  = {[0:3]};
            bins addr_mid[4]  = {[4:7]};
            bins addr_high[4] = {[8:15]};
        }

        // Coverpoint for read/write operations
        coverpoint rw {
            bins read  = {0};
            bins write = {1};
        }

        // Coverpoint for data ranges
        coverpoint data {
            bins small  = {[0:63]};
            bins medium = {[64:127]};
            bins large  = {[128:255]};
        }

        // Cross coverage: check combinations
        cross addr, rw;
        cross rw, data;

    endgroup

    // Constructor
    function new();
        trans_cov = new();
    endfunction

    function void display();
        $display("ADDR: %0d, RW: %0b, DATA: %0d", addr, rw, data);
    endfunction

endclass
```

---

## ✅ 3. Testbench to Drive Random Transactions

```systemverilog
module tb;

    logic clk;
    BusTransaction t;

    // Generate a simple clock
    initial clk = 0;
    always #5 clk = ~clk;

    initial begin
        t = new();

        repeat (30) begin
            assert(t.randomize());
            @(posedge clk);
            t.trans_cov.sample(); // sample coverage
            t.display();
        end

        $display("Simulation finished. View coverage in the simulator report.");
        $finish;
    end

endmodule
```

---

## ✅ 4. Coverage Goals

This example will help us answer:

* Did we test all parts of the address space?
* Were both read and write operations triggered?
* Did we send small, medium, and large data?
* Were all meaningful combinations of addr × rw × data covered?

---

## ✅ 5. Result Interpretation

After simulation:

* Use a coverage tool (e.g., **VCS**, **Questa**, **Xcelium**) to generate a report.
* Look for:

  * Missed bins (unexercised values)
  * Missed crosses (unexplored combinations)
  * Overall coverage percentage

---

## ✅ Summary

| Concept         | Application                                |
| --------------- | ------------------------------------------ |
| `coverpoint`    | Tracks address, read/write, data           |
| `cross`         | Measures complex behavior coverage         |
| `.sample()`     | Called after stimulus to log values        |
| Coverage report | Confirms stimulus quality and completeness |
