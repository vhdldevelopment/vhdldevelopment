#2. Flip Flops and Latches

Flip flops are single bit devices with two stable states (0 or 1) that can be used as single bit storage.  These come in a few flavours which we'll elaborate on below.  Multiple flip flops are typically connected in parallel or in series to form registers and shift-registers (more on these below).


Applications include:
* Counters
* Frequency Dividers
* Shift Registers
* Storage Registers

Latches aren't often used in FPGA and the compiler will often even throw a warning if you've implemented one as these are more common to implement accidentally than deliberately.  That said, they can often be useful if used correctly.  

!!! TODO
Link to an example of latches being used effectively.

## D-Type Flip Flop

```vhdl
library ieee;
use ieee.std_logic_1164.all;

entity D_TYPE is
  port (
      CLK   : in  std_logic;
      D     : in  std_logic;
      Q     : out std_logic
    );
end entity D_TYPE;
architecture RTL of D_TYPE is
begin
  REG_PROCESS : process(CLK)
  begin
    if rising_edge(CLK) then
      Q   <= D;
    end if;
  end process REG_PROCESS;
end architecture RTL;
```

## SR Flip Flop

```vhdl
library ieee;
use ieee.std_logic_1164.all;

entity SR_LATCH is
  port (
      CLK   : in  std_logic;
      S     : in  std_logic;
      R     : in  std_logic;
      Q     : out std_logic
    );
end entity D_TYPE;
architecture RTL of D_TYPE is
begin
  REG_PROCESS : process(CLK)
  begin
    if rising_edge(CLK) then
      if (R = '0') then
        Q <= '0';
      elsif (S = '1') then
        Q <= '1';
      end if;
    end if;
  end process REG_PROCESS;
end architecture RTL;
```

!!! TODO
    Dangers of latches.  Compile provides warning.

Applications:
* Capture short pulsed triggers


## Register

!!! Note
    Registers are typically used for small scale storage, for example storing control information for a component.  

## Shift-Register
