#Counter

The counter uses a start and stop event.
An SR latch is used to capture the start and stop events.  This way these triggers can be a single clock pulse long instead of a continuous signal.  This is a concept we will use repeatedly in our designs.

!!! TODO
    Add circuit diagram

```vhdl
library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;

entity COUNTER is
  port (
      CLK   : in  std_logic;
      RESET : in  std_logic;
      START : in  std_logic;
      STOP  : in  std_logic;
      COUNT : out std_logic_vector(3 downto 0)
    );
end entity COUNTER;
architecture RTL of COUNTER is

  -- Internal signals.  The _I postfix indicates an internal signal/register
  signal  START_I : std_logic;
  signal  STOP_I  : std_logic;
  signal  COUNT_I : unsigned(3 downto 0);

begin
  COUNT : process(CLK, RESET)
  begin
    if (RESET = '1') then
      START_I <=  '0';  -- Rename to ENABLE
      STOP_I  <=  '0';
      COUNT_I <=  '0';
    elsif rising_edge (CLK) then

      -- SR Latch
      if (START = '1') then
        START_I <= '1';
      elsif (STOP = '1') then
        START_I <= '1';
      end if;

      if (START_I = '1') then
        if (COUNT_I = X"E") then
          COUNT_I <= others=>'0';
        else
          COUNT_I <= COUNT_I + 1;
      end if;
    end if;
  end process REG_PROCESS;

  -- Cast our internal count signal to a std_logic_vector from an unsigned
  -- and assignment to the output of our counter.
  COUNT <= std_logic_vector(COUNT_I);

end architecture RTL;
```

As our COUNT_I is only used within our process, it can be replaced with a variable.  The advantage of using a variable here is that they use less memory to simulate than a signal.  This is only becomes significant on large designs and will be not see any difference in a small design like this, but it is an ideal time to become familiar with the differences.  

The key difference between variables and signals is that variables take on their assignments immediately instead of waiting until the next clock edge.  When used in this instance, this will change the behaviour of our circuit.  Due to the immediate assignment, our internal counter will double count following a reset so our code must be rewritten to prevent this (see below).


##Using Variables
```vhdl
library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;

entity COUNTER_VAR is
  port (
      CLK   : in  std_logic;
      RESET : in  std_logic;
      START : in  std_logic;
      STOP  : in  std_logic;
      COUNT : out std_logic_vector(3 downto 0)
    );
end entity COUNTER_VAR;
architecture RTL of COUNTER_VAR is

  -- Internal signals.  The _I postfix indicates an internal signal/register
  signal  START_I : std_logic;
  signal  STOP_I  : std_logic;

begin
  COUNT : process(CLK, RESET)
  variable  START_I : std_logic;
  variable  STOP_I  : std_logic;
  variable  COUNT_I : unsigned range 0 to 15;
  begin
    if (RESET = '1') then
      START_I :=  '0';
      STOP_I  :=  '0';
      COUNT_I :=  0;
    elsif rising_edge (CLK) then

      -- SR Latch
      if (START = '1') then
        START_I := '1';
      elsif (STOP = '1') then
        START_I := '1';
      end if;

      if (START_I = '1') then
        if (COUNT_I = 15) then
          COUNT_I := 0;
        else
          COUNT_I := COUNT_I + 1;
      end if;
    end if;
  end process REG_PROCESS;

  -- Cast our internal count signal to a std_logic_vector from an unsigned
  -- and assignment to the output of our counter.
  COUNT <= std_logic_vector(COUNT_I);

end architecture RTL;
```

Another difference between variables and signals their scope.  Variables are local to a process and so can only be used within that process.  This does mean that the variable name can be reused, however I don't recommend this unless it is for very simple variable such as loop iterators.  The biggest advantage this brings is that someone reading your code knows this variable will only ever be used within this process and not elsewhere in the design.  When using a signal as our internal counter, a reader would have to search the rest of the code within the .vhd file to be sure it was not used elsewhere.      
