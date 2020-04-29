# 4. State Machines

Design assumes inputs are synchronized to the FPGA's clock.
Otherwise the inputs would need to be clocked through an input register.
Some modules will do this as standard procedure if the additional latency will not cause an issue.

!!! TODO
    Recreate the circuit diagram

Waits in abort state until the abort signal goes low for improved stability.
Complete

```vhdl
    library ieee;
    use ieee.std_logic_1164.all;
    use ieee.numeric_std.all;

    entity STATE_MACHINE is
      port (
        CLK       : in  std_logic;
        RESET     : in  std_logic;
        START     : in  std_logic;
        STOP      : in  std_logic;
        COMPLETE  : out std_logic
      );
    end entity STATE_MACHINE;
    architecture RTL of STATE_MACHINE is

      type T_STATES is (
          IDLE,
          ACTIVE,
          FINISH,
          ABORT
      );

      signal  STATE_REG : T_STATES;
      signal  COUNT     : unsigned(7 downto 0);
      signal  EN        : std_logic;
      signal  CLR       : std_logic;

    begin

      STATE_PROC  : process (CLK, RESET)
      begin
        if (RESET = '1') then
          STATE_REG <=  IDLE;
        elsif (rising_edge(CLK)) then
          case (STATE_REG) is
            when IDLE =>
              EN  <= '0';
              CLR <= '0';
              COMPLETE <= '0';
              if (START = '1') then
                STATE_REG <= ACTIVE;
              end if;
            when ACTIVE =>
              EN  <= '1';
              CLR <= '0';
              COMPLETE <= '0';
              if (ABORT = '1') then
                STATE_REG <= ABORT;
              elsif (COUNT = X"20") then
                STATE_REG <= FINISH;
              end if;
            when FINISH =>
              EN  <= '0';
              CLR <= '1';
              COMPLETE <= '1';            
              STATE_REG <= IDLE;
            when ABORT =>
              if (ABORT = '0') then
                EN  <= '0';
                CLR <= '1';
                COMPLETE <= '0';
                STATE_REG <= IDLE;
              end if;
            when others =>
          end case;
        end if;
      end process;

      COUNT_PROC  : process (CLK, RESET)
      begin
        if (RESET = '1') then
          COUNT <= (others=> '0');
        elsif (rising_edge(CLK)) then
          if (CLR = '1') then
            COUNT <= (others=> '0');
          elsif (EN = '1') then
            COUNT <=  COUNT + 1;
          end if;
        end if;
      end process;

    end architecture RTL;
```
