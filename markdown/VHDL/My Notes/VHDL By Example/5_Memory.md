# 5. Memory

Memory can be defined in three ways:

1. Infer the memory directly via the VHDL code
    * This is the most portable and flexible option but also requires the most design effort and is likely to be the least efficient.  We may choose to use this less efficient option if our design is smaller than the smallest available primitive RAM block and save these primitives for where they are really needed.
2. Build the memory using the vendor's primitive RAM structures
    * This will be more efficient than the first option as it will use RAM resources built into the FPGA.  It will save design effort too as we will now only need to code the control structures around the primitive.
3. Design the memory using the FPGA vendor's specialised tools (easiest but least portable).
    * This is the easiest option in terms of design effort but does come with a heavily reliance on the vendor.  If the design is mitigated to different device in future these IP cores will need to be completely redesigned.  This may also be an issue if our code needs to be supported for a long time.  Any changes to the vendor's tool may have a direct impact on our design, or worse still the vendor may choose to make this tool obsolete entirely which may force a complete redesign within an existing product.  Obviously this may never be an issue, but does come with a rather large headache if it does.

Below we implement the same memory structure using option 1 (directly via VHDL code) with two different access methods, single port and dual port.
The dual port design has the advantage that read and write access is available simultaneously (or two simultaneous writes).  It also provides a method of moving data across clock domains.

```vhdl
library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;

entity SINGLE_PORT_RAM is
  port (
    CLK       : in  std_logic;
    DATA_IN   : in  std_logic_vector(15 downto 0);
    WR_ADDR   : in  std_logic_vector(9 downto 0);
    RD_ADDR   : in  std_logic_vector(9 downto 0);
    WR_EN     : in  std_logic;
    DATA_OUT  : out std_logic_vector(15 downto 0)
  );
end entity SINGLE_PORT_RAM;
architecture RTL of SINGLE_PORT_RAM is
  type MEMBLK_1Kx16 is array(1023 downto 0) of std_logic_vector(15 downto 0);
  signal RAM : MEMBLK_1Kx16
begin
  RAM_PROC : process(CLK)
  begin
    if rising_edge(CLK) then
      if(WR_EN = '1') then
        RAM( to_integer(unsigned(WR_ADDR)) ) <= DATA_IN;
        DATA_OUT <= DATA_IN;
      else
        DATA_OUT <= RAM( to_integer(unsigned(RD_ADDR)) );
      end if;
    end if;
  end process RAM_PROC;
end architecture RTL;
```

```vhdl
library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;

entity DUAL_PORT_RAM is
  port (
    -- Port A
    CLK_A       : in  std_logic;
    WR_EN_A     : in  std_logic;
    ADDR_A      : in  std_logic_vector(9 downto 0);
    DATA_IN_A   : in  std_logic_vector(15 downto 0);
    DATA_OUT_A  : out std_logic_vector(15 downto 0);
    -- Port B
    CLK_B       : in  std_logic;
    WR_EN_B     : in  std_logic;
    ADDR_B      : in  std_logic_vector(9 downto 0);
    DATA_IN_B   : in  std_logic_vector(15 downto 0);
    DATA_OUT_B  : out std_logic_vector(15 downto 0)
  );
end entity DUAL_PORT_RAM;
architecture RTL of DUAL_PORT_RAM is
  type MEMBLK_1Kx16 is array(1023 downto 0) of std_logic_vector(15 downto 0);
  signal RAM : MEMBLK_1Kx16
begin
  RAM_PROC_A : process(CLK_A)
  begin
    if rising_edge(CLK_A) then
      if(WR_EN_A = '1') then
        RAM( to_integer(unsigned(WR_ADDR_A)) ) <= DATA_IN_A;
        DATA_OUT_A <= DATA_IN_A;
      else
        DATA_OUT_A <= RAM( to_integer(unsigned(ADDR_A)) );
      end if;
    end if;
  end process RAM_PROC_A;

  RAM_PROC_B : process(CLK_B)
  begin
    if rising_edge(CLK_B) then
      if(WR_EN_B = '1') then
        RAM( to_integer(unsigned(WR_ADDR_B)) ) <= DATA_IN_B;
        DATA_OUT_B <= DATA_IN_B;
      else
        DATA_OUT_B <= RAM( to_integer(unsigned(ADDR_B)) );
      end if;
    end if;
  end process RAM_PROC_B;  
end architecture RTL;
```

In our dual port design, we have no mechanism within VHDL to define the behaviour when the same address is accessed by both ports simultaneously.  The behaviour will be determined by  the synthesis software or the hardware fabric itself.
