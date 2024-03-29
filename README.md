Designed a basketball scoreboard using VHDL on a Basys 3 FPGA board and implementied it in Vivado.


--the code implements a functional scoreboard system with the ability to count scores for two teams, handle reset operations, prevent accidental score changes, and display the scores on seven-segment displays. The use of a state machine ensures proper control flow, and debounce logic enhances the reliability of input signals.
--file  Scoreboard.vhd

library IEEE;
use IEEE.STD_LOGIC_1164.ALL;

-- Uncomment the following library declaration if using
-- arithmetic functions with Signed or Unsigned values
use IEEE.NUMERIC_STD.ALL;

-- Uncomment the following library declaration if instantiating
-- any Xilinx leaf cells in this code.
--library UNISIM;
--use UNISIM.VComponents.all;

entity score is
    Port ( rst1 : in STD_LOGIC;
           clk : in STD_LOGIC;
           button : in STD_LOGIC_VECTOR (0 to 3);
           seg : out STD_LOGIC_VECTOR (0 to 6);
           dp : out STD_LOGIC;
           an : out STD_LOGIC_VECTOR (3 downto 0));
end score;

architecture Behavioral of score is
type states is (start, press, count_up1_team1, count_up2_team1, count_up1_team2, count_up2_team2);                                                                             
signal current_state, next_state : states;                                                                                                                         
signal i : integer range 0 to 3;                                                                                                                                   
constant n : integer := 5*10**6;                                                                                                                             
signal divider : integer := 0;                                                                                                                                     
signal ce : std_logic := '0';     

component Scoreboard_file2 is                                                                                                                                            
    Port ( clk : in STD_LOGIC;                                                                            
           Din : in STD_LOGIC_VECTOR (15 downto 0);                                                          
           an : out STD_LOGIC_VECTOR (3 downto 0);                                                         
           seg : out STD_LOGIC_VECTOR (0 to 6);                                                       
           dp_in : in STD_LOGIC_VECTOR (3 downto 0);                                                                                
           dp_out : out STD_LOGIC;                                                                                 
           rst : in STD_LOGIC);                                                                                                               
end component Scoreboard_file2; 

component deBounce1 is
    port(   clk : in std_logic;
            rst : in std_logic;
            button_in : in std_logic;
            pulse_out : out std_logic
        );
end component DeBounce1;                                                                                                                                         
                                                                                                                                                                                                        
signal rst : std_logic; 
--signal score_team1, score_team2 : integer range 0 to 99 := 0;                                                                             
signal scorafisat:std_logic_vector(15 downto 0);

begin 
RESET: deBounce1 port map (clk => clk, rst => '0', button_in => rst1, pulse_out => rst);
                                                                                                                                                                   
-- divider                                                                                                                                               
process (rst, clk)                                                                                                                                                 
                                                                                                                                                                   
begin                                                                                                                                                              
    if rst = '1' then                                                                                                                                              
        divider <= 0;                                                                                                                                              
        ce <= '0';                                                                                                                                                 
    elsif rising_edge(clk) then                                                                                                                                    
        if divider = n - 1 then                                                                                                                                    
            divider <= 0;                                                                                                                                          
            ce <= '1';                                                                                                                                             
        else                                                                                                                                                       
            divider <= divider + 1;                                                                                                                                
            ce <= '0';                                                                                                                                             
        end if;                                                                                                                                                    
    end if;                                                                                                                                                        
end process;                                                                                                                                                       
                                                                                                                                                                   
ff : process(rst, clk)                                                                                                                                             
begin                                                                                                                                                              
  if rst = '1' then                                                                                                                                                
    current_state <= start;                                                                                                                                        
  elsif rising_edge(clk) then                                                                                                                                      
    current_state <= next_state;                                                                                                                                   
  end if;                                                                                                                                                          
end process;

clc : process (current_state, button, i)
    begin
        case current_state is
            when start =>
                next_state <= press;
            when press =>
                if button(0) = '1' then
                    next_state <= count_up1_team1;
                elsif button(1) = '1' then
                    next_state <= count_up2_team1;
                elsif button(2) = '1' then
                    next_state <= count_up1_team2;
                elsif button(3) = '1' then
                    next_state <= count_up2_team2;
                elsif button(i) = '0' then
                    next_state <= press;
                end if;
            when count_up1_team1 =>
                next_state <= start;
            when count_up2_team1 =>
                next_state <= start;
            when count_up1_team2 =>
                next_state <= start;
            when count_up2_team2 =>
                next_state <= start;
            when others =>
                next_state <= start;
        end case;
    end process;

generare_score:process(clk,rst)
 variable z1, u1, z2, u2 : integer range 0 to 9 := 0;
begin
   if rst = '1' then
       z1 := 0;
       u1 := 0;
       z2 := 0;
       u2 := 0;
   elsif rising_edge(clk) then
       if ce = '1' then
           if current_state = count_up1_team1 then
               if u1 = 9 then
                   u1 := 0;
                   z1 := z1 + 1;
                   end if;
               else
                   u1 := u1 + 1;
               end if;
           end if;
           if current_state = count_up2_team1 then
               if u1 = 8 then
                   u1 := 0;
                   z1 := z1 + 1;
                end if;
                if u1 = 9 then
                   u1 := 1;
                   z1 := z1 + 1;
                 end if;
               else
                   u1 := u1 + 2;
               end if;
           end if;
           if current_state = count_up1_team2 then
               if u2 = 9 then
                     u2 := 0;
                     z2 := z2 + 1;
               else
                  u2 := u2 + 1;
            end if;
            if current_state = count_up2_team2 then
                          if u2 = 9 then
                              u2 := 1;
                              z2 := z2 + 1;
                              end if;
                           if u2 = 8 then
                              u2 := 0;
                              z2 := z2 + 1;
                               end if;
                          else
                              u2 := u2 + 2;
                          end if;
                      end if;
--                score_team1 <= z1 * 10 + u1; 
--                score_team2 <= z2 * 10 + u2;


scorafisat <= std_logic_vector(to_unsigned(z1,4)) &
              std_logic_vector(to_unsigned(u1,4)) &
              std_logic_vector(to_unsigned(z2,4)) &
              std_logic_vector(to_unsigned(u2,4));

end process; 

display :   Scoreboard_file2 port map (
                                     clk => clk,
                                     Din => scorafisat,
                                     an => an,
                                     seg => seg,
                                     dp_in => (others => '0'),
                                     dp_out => dp, 
                                     rst => rst);
                                                                                                                                                                                                                                                                                
end Behavioral;


--this module efficiently handles the display multiplexing, data selection, and seven-segment decoding to output the scores on the seven-segment displays according to the input data and control signals. It utilizes frequency division techniques to ensure proper synchronization of the display multiplexing process.

--file  Scoreboard_file2.vhd

library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.STD_LOGIC_UNSIGNED.ALL;

entity Scoreboard_file2 is
    Port ( clk : in STD_LOGIC; --100MHz board clock input
           Din : in STD_LOGIC_VECTOR (15 downto 0); --16 bit binary data for 4 displays
           an : out STD_LOGIC_VECTOR (3 downto 0); --anode outputs selecting individual displays 3 to 0
           seg : out STD_LOGIC_VECTOR (0 to 6); -- cathode outputs for selecting LED-s in each display
           dp_in : in STD_LOGIC_VECTOR (3 downto 0); --decimal point input values
           dp_out : out STD_LOGIC; --selected decimal point sent to cathodes
           rst : in STD_LOGIC); --global reset
end Scoreboard_file2;

architecture Behavioral of Scoreboard_file2 is

signal clk1kHz : STD_LOGIC;
signal state : STD_LOGIC_VECTOR(16 downto 0);
signal addr : STD_LOGIC_VECTOR(1 downto 0);
signal cseg : STD_LOGIC_VECTOR(3 downto 0);

begin

-- frequency divider by 100k to generate 1kHz anode sweeping clock
-- counting from 0 to 99999, output is MSB 
-- 17 counter state length needed 
div1kHz: process(clk, rst)
begin
   if rst = '1' then 
        state <= '0' & X"0000";
   else
     if rising_edge(clk) then
        if state = '1' & X"869F" then --if counte reaches 99999
            state <= '0' & X"0000"; -- reset back to 0
        else
            state <= state+1;
        end if;
     end if;
   end if;         
end process;

clk1Khz <= state(16); --assign MSB to frequency divider output


-- 2 bit counter generating 4 addresses for display multiplexing
counter_2bits: process(clk1kHz)
begin
  if rising_edge(clk1kHz) then       
           addr <= addr+1;   
  end if;   
end process;

-- 2 to 4 decoder used to select one display of 4 at each sweeping address generated by the 2 bit counter 
-- anodes are active low, decoder must provide '0' for activation
dcd3_8: process(addr)
begin
  case addr is
      when "00" =>  an <= "0111";       
      when "01" =>  an <= "1011"; 
      when "10" =>  an <= "1101"; 
      when "11" =>  an <= "1110"; 
      when others => an <= "1111";
   end case; 
end process;

--4 input multiplexer to select data to be sent to a single display
--synchronzied with addr and display activation with the anodes
data_mux4: process(addr,Din,dp_in)
begin
  case addr is
      when "00" =>  cseg <= Din(15 downto 12); --sending 4 upper bits targeted at display 3  
                    dp_out <= not dp_in(3); -- lighting up decimal point on display 3
      when "01" =>  cseg <= Din(11 downto 8); --sending next 4 bits targeted at display 2
                    dp_out <= not dp_in(2); -- lighting up decimal point on display 2
      when "10" =>  cseg <= Din(7 downto 4);  -- ....
                    dp_out <= not dp_in(1);
      when "11" =>  cseg <= Din(3 downto 0); -- ....
                    dp_out <= not dp_in(0);
      when others => cseg <= "XXXX";
                     dp_out <= 'X';
   end case; 
end process;

--binary to 7 segment decoder
--cathodes also active low, provide '0' for a lit up segment or decimal point
dcd7seg:process(cseg)
begin
  case cseg is
      when "0000" =>  seg <= "0000001"; 
      when "0001" =>  seg <= "1001111"; 
      when "0010" =>  seg <= "0010010"; 
      when "0011" =>  seg <= "0000110"; 
      when "0100" =>  seg <= "1001100"; 
      when "0101" =>  seg <= "0100100"; 
      when "0110" =>  seg <= "0100000"; 
      when "0111" =>  seg <= "0001111";
      when "1000" =>  seg <= "0000000"; 
      when "1001" =>  seg <= "0000100"; 
      when "1010" =>  seg <= "0000010"; 
      when "1011" =>  seg <= "1100000"; 
      when "1100" =>  seg <= "0110001"; 
      when "1101" =>  seg <= "1000010"; 
      when "1110" =>  seg <= "0110000"; 
      when "1111" =>  seg <= "0111000";
      when others => seg <= "XXXXXXX";
  end case; 
      end process;
      
      end Behavioral;

-- This debounce module ensures that the output pulse signal remains stable for a certain period after detecting a change in the input button signal
--file  deBounce1.vhd

library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL;

entity deBounce1 is
    port(   clk : in std_logic;
            rst : in std_logic;
            button_in : in std_logic;
            pulse_out : out std_logic
        );
end DeBounce1;

architecture behav of deBounce1 is

--the below constants decide the working parameters.
--the higher this is, the more longer time the user has to press the button.
constant COUNT_MAX : integer := 5*10**8;
--set it '1' if the button creates a high pulse when its pressed, otherwise '0'.
constant BTN_ACTIVE : std_logic := '1';

signal count : integer := 0;
type state_type is (idle,wait_time); --state machine
signal state : state_type := idle;

begin
  
process(rst,clk)
begin
    if(rst = '1') then
        state <= idle;
        pulse_out <= '0';
   elsif(rising_edge(clk)) then
        case (state) is
            when idle =>
                if(button_in = BTN_ACTIVE) then  
                    state <= wait_time;
                else
                    state <= idle; --wait until button is pressed.
                end if;
                pulse_out <= '0';
            when wait_time =>
                if(count = COUNT_MAX) then
                    count <= 0;
                    if(button_in = BTN_ACTIVE) then
                        pulse_out <= '1';
                    end if;
                    state <= idle;  
                else
                    count <= count + 1;
                end if; 
        end case;       
    end if;        
end process;                  
                                                                                
end architecture behav;

--This file acts as a bridge between the logical design implemented in HDL (Hardware Description Language) and the physical hardware of the FPGA device

--file   const.xdc

## This file is a general .xdc for the Basys3 rev B board                                                                                               
## To use it in a project:                                                                                                                              
## - uncomment the lines corresponding to used pins                                                                                                     
## - rename the used ports (in each line, after get_ports) according to the top level signal names in the project                                       
                                                                                                                                                        
## Clock signal                                                                                                                                         
set_property -dict { PACKAGE_PIN W5   IOSTANDARD LVCMOS33 } [get_ports clk]                                                                             
create_clock -add -name sys_clk_pin -period 10.00 -waveform {0 5} [get_ports clk]                                                                       
                                                                                                                                                        
                                                                                                                                                        
## Switches                                                                                                                                             
#set_property -dict { PACKAGE_PIN V17   IOSTANDARD LVCMOS33 } [get_ports {sw[0]}]                                                                       
#set_property -dict { PACKAGE_PIN V16   IOSTANDARD LVCMOS33 } [get_ports {sw[1]}]                                                                       
#set_property -dict { PACKAGE_PIN W16   IOSTANDARD LVCMOS33 } [get_ports {sw[2]}]                                                                       
#set_property -dict { PACKAGE_PIN W17   IOSTANDARD LVCMOS33 } [get_ports {sw[3]}]                                                                       
#set_property -dict { PACKAGE_PIN W15   IOSTANDARD LVCMOS33 } [get_ports {sw[4]}]                                                                       
#set_property -dict { PACKAGE_PIN V15   IOSTANDARD LVCMOS33 } [get_ports {sw[5]}]                                                                       
#set_property -dict { PACKAGE_PIN W14   IOSTANDARD LVCMOS33 } [get_ports {sw[6]}]                                                                       
#set_property -dict { PACKAGE_PIN W13   IOSTANDARD LVCMOS33 } [get_ports {sw[7]}]                                                                       
#set_property -dict { PACKAGE_PIN V2    IOSTANDARD LVCMOS33 } [get_ports {sw[8]}]                                                                       
#set_property -dict { PACKAGE_PIN T3    IOSTANDARD LVCMOS33 } [get_ports {sw[9]}]                                                                       
#set_property -dict { PACKAGE_PIN T2    IOSTANDARD LVCMOS33 } [get_ports {sw[10]}]                                                                      
#set_property -dict { PACKAGE_PIN R3    IOSTANDARD LVCMOS33 } [get_ports {sw[11]}]                                                                      
#set_property -dict { PACKAGE_PIN W2    IOSTANDARD LVCMOS33 } [get_ports {sw[12]}]                                                                      
#set_property -dict { PACKAGE_PIN U1    IOSTANDARD LVCMOS33 } [get_ports {sw[13]}]                                                                      
#set_property -dict { PACKAGE_PIN T1    IOSTANDARD LVCMOS33 } [get_ports {sw[14]}]                                                                      
#set_property -dict { PACKAGE_PIN R2    IOSTANDARD LVCMOS33 } [get_ports {sw[15]}]                                                                      
                                                                                                                                                        
                                                                                                                                                        
## LEDs                                                                                                                                                 
#set_property -dict { PACKAGE_PIN U16   IOSTANDARD LVCMOS33 } [get_ports {led[0]}]                                                                      
#set_property -dict { PACKAGE_PIN E19   IOSTANDARD LVCMOS33 } [get_ports {led[1]}]                                                                      
#set_property -dict { PACKAGE_PIN U19   IOSTANDARD LVCMOS33 } [get_ports {led[2]}]                                                                      
#set_property -dict { PACKAGE_PIN V19   IOSTANDARD LVCMOS33 } [get_ports {led[3]}]                                                                      
#set_property -dict { PACKAGE_PIN W18   IOSTANDARD LVCMOS33 } [get_ports {led[4]}]                                                                      
#set_property -dict { PACKAGE_PIN U15   IOSTANDARD LVCMOS33 } [get_ports {led[5]}]                                                                      
#set_property -dict { PACKAGE_PIN U14   IOSTANDARD LVCMOS33 } [get_ports {led[6]}]                                                                      
#set_property -dict { PACKAGE_PIN V14   IOSTANDARD LVCMOS33 } [get_ports {led[7]}]                                                                      
#set_property -dict { PACKAGE_PIN V13   IOSTANDARD LVCMOS33 } [get_ports {led[8]}]                                                                      
#set_property -dict { PACKAGE_PIN V3    IOSTANDARD LVCMOS33 } [get_ports {led[9]}]                                                                      
#set_property -dict { PACKAGE_PIN W3    IOSTANDARD LVCMOS33 } [get_ports {led[10]}]                                                                     
#set_property -dict { PACKAGE_PIN U3    IOSTANDARD LVCMOS33 } [get_ports {led[11]}]                                                                     
#set_property -dict { PACKAGE_PIN P3    IOSTANDARD LVCMOS33 } [get_ports {led[12]}]                                                                     
#set_property -dict { PACKAGE_PIN N3    IOSTANDARD LVCMOS33 } [get_ports {led[13]}]                                                                     
#set_property -dict { PACKAGE_PIN P1    IOSTANDARD LVCMOS33 } [get_ports {led[14]}]                                                                     
#set_property -dict { PACKAGE_PIN L1    IOSTANDARD LVCMOS33 } [get_ports {led[15]}]                                                                     
                                                                                                                                                        
                                                                                                                                                        
##7 Segment Display                                                                                                                                     
set_property -dict { PACKAGE_PIN W7   IOSTANDARD LVCMOS33 } [get_ports {seg[0]}]                                                                        
set_property -dict { PACKAGE_PIN W6   IOSTANDARD LVCMOS33 } [get_ports {seg[1]}]                                                                        
set_property -dict { PACKAGE_PIN U8   IOSTANDARD LVCMOS33 } [get_ports {seg[2]}]                                                                        
set_property -dict { PACKAGE_PIN V8   IOSTANDARD LVCMOS33 } [get_ports {seg[3]}]                                                                        
set_property -dict { PACKAGE_PIN U5   IOSTANDARD LVCMOS33 } [get_ports {seg[4]}]                                                                        
set_property -dict { PACKAGE_PIN V5   IOSTANDARD LVCMOS33 } [get_ports {seg[5]}]                                                                        
set_property -dict { PACKAGE_PIN U7   IOSTANDARD LVCMOS33 } [get_ports {seg[6]}]                                                                        
                                                                                                                                                        
set_property -dict { PACKAGE_PIN V7   IOSTANDARD LVCMOS33 } [get_ports dp]                                                                              
                                                                                                                                                        
set_property -dict { PACKAGE_PIN U2   IOSTANDARD LVCMOS33 } [get_ports {an[0]}]                                                                         
set_property -dict { PACKAGE_PIN U4   IOSTANDARD LVCMOS33 } [get_ports {an[1]}]                                                                         
set_property -dict { PACKAGE_PIN V4   IOSTANDARD LVCMOS33 } [get_ports {an[2]}]                                                                         
set_property -dict { PACKAGE_PIN W4   IOSTANDARD LVCMOS33 } [get_ports {an[3]}]                                                                         
                                                                                                                                                        
                                                                                                                                                        
##Buttons                                                                                                                                               
set_property -dict { PACKAGE_PIN U18   IOSTANDARD LVCMOS33 } [get_ports rst1]                                                                            
set_property -dict { PACKAGE_PIN T18   IOSTANDARD LVCMOS33 } [get_ports button[0]]                                                                      
set_property -dict { PACKAGE_PIN W19   IOSTANDARD LVCMOS33 } [get_ports button[3]]                                                                      
set_property -dict { PACKAGE_PIN T17   IOSTANDARD LVCMOS33 } [get_ports button[2]]                                                                      
set_property -dict { PACKAGE_PIN U17   IOSTANDARD LVCMOS33 } [get_ports button[1]]                                                                      
                                                                                                                                                        
                                                                                                                                                        
##Pmod Header JA                                                                                                                                        
#set_property -dict { PACKAGE_PIN J1   IOSTANDARD LVCMOS33 } [get_ports {JA[0]}];#Sch name = JA1                                                        
#set_property -dict { PACKAGE_PIN L2   IOSTANDARD LVCMOS33 } [get_ports {JA[1]}];#Sch name = JA2                                                        
#set_property -dict { PACKAGE_PIN J2   IOSTANDARD LVCMOS33 } [get_ports {JA[2]}];#Sch name = JA3                                                        
#set_property -dict { PACKAGE_PIN G2   IOSTANDARD LVCMOS33 } [get_ports {JA[3]}];#Sch name = JA4                                                        
#set_property -dict { PACKAGE_PIN H1   IOSTANDARD LVCMOS33 } [get_ports {JA[4]}];#Sch name = JA7                                                        
#set_property -dict { PACKAGE_PIN K2   IOSTANDARD LVCMOS33 } [get_ports {JA[5]}];#Sch name = JA8                                                        
#set_property -dict { PACKAGE_PIN H2   IOSTANDARD LVCMOS33 } [get_ports {JA[6]}];#Sch name = JA9                                                        
#set_property -dict { PACKAGE_PIN G3   IOSTANDARD LVCMOS33 } [get_ports {JA[7]}];#Sch name = JA10                                                       
                                                                                                                                                        
##Pmod Header JB                                                                                                                                        
#set_property -dict { PACKAGE_PIN A14   IOSTANDARD LVCMOS33 } [get_ports {JB[0]}];#Sch name = JB1                                                       
#set_property -dict { PACKAGE_PIN A16   IOSTANDARD LVCMOS33 } [get_ports {JB[1]}];#Sch name = JB2                                                       
#set_property -dict { PACKAGE_PIN B15   IOSTANDARD LVCMOS33 } [get_ports {JB[2]}];#Sch name = JB3                                                       
#set_property -dict { PACKAGE_PIN B16   IOSTANDARD LVCMOS33 } [get_ports {JB[3]}];#Sch name = JB4                                                       
#set_property -dict { PACKAGE_PIN A15   IOSTANDARD LVCMOS33 } [get_ports {JB[4]}];#Sch name = JB7                                                       
#set_property -dict { PACKAGE_PIN A17   IOSTANDARD LVCMOS33 } [get_ports {JB[5]}];#Sch name = JB8                                                       
#set_property -dict { PACKAGE_PIN C15   IOSTANDARD LVCMOS33 } [get_ports {JB[6]}];#Sch name = JB9                                                       
#set_property -dict { PACKAGE_PIN C16   IOSTANDARD LVCMOS33 } [get_ports {JB[7]}];#Sch name = JB10                                                      
                                                                                                                                                        
##Pmod Header JC                                                                                                                                        
#set_property -dict { PACKAGE_PIN K17   IOSTANDARD LVCMOS33 } [get_ports {JC[0]}];#Sch name = JC1                                                       
#set_property -dict { PACKAGE_PIN M18   IOSTANDARD LVCMOS33 } [get_ports {JC[1]}];#Sch name = JC2                                                       
#set_property -dict { PACKAGE_PIN N17   IOSTANDARD LVCMOS33 } [get_ports {JC[2]}];#Sch name = JC3                                                       
#set_property -dict { PACKAGE_PIN P18   IOSTANDARD LVCMOS33 } [get_ports {JC[3]}];#Sch name = JC4                                                       
#set_property -dict { PACKAGE_PIN L17   IOSTANDARD LVCMOS33 } [get_ports {JC[4]}];#Sch name = JC7                                                       
#set_property -dict { PACKAGE_PIN M19   IOSTANDARD LVCMOS33 } [get_ports {JC[5]}];#Sch name = JC8                                                       
#set_property -dict { PACKAGE_PIN P17   IOSTANDARD LVCMOS33 } [get_ports {JC[6]}];#Sch name = JC9                                                       
#set_property -dict { PACKAGE_PIN R18   IOSTANDARD LVCMOS33 } [get_ports {JC[7]}];#Sch name = JC10                                                      
                                                                                                                                                        
##Pmod Header JXADC                                                                                                                                     
#set_property -dict { PACKAGE_PIN J3   IOSTANDARD LVCMOS33 } [get_ports {JXADC[0]}];#Sch name = XA1_P                                                   
#set_property -dict { PACKAGE_PIN L3   IOSTANDARD LVCMOS33 } [get_ports {JXADC[1]}];#Sch name = XA2_P                                                   
#set_property -dict { PACKAGE_PIN M2   IOSTANDARD LVCMOS33 } [get_ports {JXADC[2]}];#Sch name = XA3_P                                                   
#set_property -dict { PACKAGE_PIN N2   IOSTANDARD LVCMOS33 } [get_ports {JXADC[3]}];#Sch name = XA4_P                                                   
#set_property -dict { PACKAGE_PIN K3   IOSTANDARD LVCMOS33 } [get_ports {JXADC[4]}];#Sch name = XA1_N                                                   
#set_property -dict { PACKAGE_PIN M3   IOSTANDARD LVCMOS33 } [get_ports {JXADC[5]}];#Sch name = XA2_N                                                   
#set_property -dict { PACKAGE_PIN M1   IOSTANDARD LVCMOS33 } [get_ports {JXADC[6]}];#Sch name = XA3_N                                                   
#set_property -dict { PACKAGE_PIN N1   IOSTANDARD LVCMOS33 } [get_ports {JXADC[7]}];#Sch name = XA4_N                                                   
                                                                                                                                                        
                                                                                                                                                        
##VGA Connector                                                                                                                                         
#set_property -dict { PACKAGE_PIN G19   IOSTANDARD LVCMOS33 } [get_ports {vgaRed[0]}]                                                                   
#set_property -dict { PACKAGE_PIN H19   IOSTANDARD LVCMOS33 } [get_ports {vgaRed[1]}]                                                                   
#set_property -dict { PACKAGE_PIN J19   IOSTANDARD LVCMOS33 } [get_ports {vgaRed[2]}]                                                                   
#set_property -dict { PACKAGE_PIN N19   IOSTANDARD LVCMOS33 } [get_ports {vgaRed[3]}]                                                                   
#set_property -dict { PACKAGE_PIN N18   IOSTANDARD LVCMOS33 } [get_ports {vgaBlue[0]}]                                                                  
#set_property -dict { PACKAGE_PIN L18   IOSTANDARD LVCMOS33 } [get_ports {vgaBlue[1]}]                                                                  
#set_property -dict { PACKAGE_PIN K18   IOSTANDARD LVCMOS33 } [get_ports {vgaBlue[2]}]                                                                  
#set_property -dict { PACKAGE_PIN J18   IOSTANDARD LVCMOS33 } [get_ports {vgaBlue[3]}]                                                                  
#set_property -dict { PACKAGE_PIN J17   IOSTANDARD LVCMOS33 } [get_ports {vgaGreen[0]}]                                                                 
#set_property -dict { PACKAGE_PIN H17   IOSTANDARD LVCMOS33 } [get_ports {vgaGreen[1]}]                                                                 
#set_property -dict { PACKAGE_PIN G17   IOSTANDARD LVCMOS33 } [get_ports {vgaGreen[2]}]                                                                 
#set_property -dict { PACKAGE_PIN D17   IOSTANDARD LVCMOS33 } [get_ports {vgaGreen[3]}]                                                                 
#set_property -dict { PACKAGE_PIN P19   IOSTANDARD LVCMOS33 } [get_ports Hsync]                                                                         
#set_property -dict { PACKAGE_PIN R19   IOSTANDARD LVCMOS33 } [get_ports Vsync]                                                                         
                                                                                                                                                        
                                                                                                                                                        
##USB-RS232 Interface                                                                                                                                   
#set_property -dict { PACKAGE_PIN B18   IOSTANDARD LVCMOS33 } [get_ports RsRx]                                                                          
#set_property -dict { PACKAGE_PIN A18   IOSTANDARD LVCMOS33 } [get_ports RsTx]                                                                          
                                                                                                                                                        
                                                                                                                                                        
##USB HID (PS/2)                                                                                                                                        
#set_property -dict { PACKAGE_PIN C17   IOSTANDARD LVCMOS33   PULLUP true } [get_ports PS2Clk]                                                          
#set_property -dict { PACKAGE_PIN B17   IOSTANDARD LVCMOS33   PULLUP true } [get_ports PS2Data]                                                         
                                                                                                                                                        
                                                                                                                                                        
##Quad SPI Flash                                                                                                                                        
##Note that CCLK_0 cannot be placed in 7 series devices. You can access it using the                                                                    
##STARTUPE2 primitive.                                                                                                                                  
#set_property -dict { PACKAGE_PIN D18   IOSTANDARD LVCMOS33 } [get_ports {QspiDB[0]}]                                                                   
#set_property -dict { PACKAGE_PIN D19   IOSTANDARD LVCMOS33 } [get_ports {QspiDB[1]}]                                                                   
#set_property -dict { PACKAGE_PIN G18   IOSTANDARD LVCMOS33 } [get_ports {QspiDB[2]}]                                                                   
#set_property -dict { PACKAGE_PIN F18   IOSTANDARD LVCMOS33 } [get_ports {QspiDB[3]}]                                                                   
#set_property -dict { PACKAGE_PIN K19   IOSTANDARD LVCMOS33 } [get_ports QspiCSn]                                                                       
                                                                                                                                                        
                                                                                                                                                        
## Configuration options, can be used for all designs                                                                                                   
set_property CONFIG_VOLTAGE 3.3 [current_design]                                                                                                        
set_property CFGBVS VCCO [current_design]                                                                                                               
                                                                                                                                                        
## SPI configuration mode options for QSPI boot, can be used for all designs                                                                            
set_property BITSTREAM.GENERAL.COMPRESS TRUE [current_design]                                                                                           
set_property BITSTREAM.CONFIG.CONFIGRATE 33 [current_design]                                                                                            
set_property CONFIG_MODE SPIx4 [current_design]
