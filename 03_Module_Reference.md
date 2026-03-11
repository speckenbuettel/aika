# Timberwolf Server -- Module Reference

This file contains the canonical reference of all Custom Logic modules
of the Timberwolf Server. It defines the valid syntax and serves as the
authoritative source for module parameters, inputs, outputs and
behavior.

------------------------------------------------------------------------

# Module Index

Polynomial Ratio Limiter Comparator And Or Xor Clocksignal Monoflop
Latch Multiplexer JsonPathSelector HEX-\>INT STR-\>FLOAT STR-\>INT
Printf Regex Stopwatch PID controller PID_awu Statemachine Statistic
Ramp Astro Localtime Wakeup SendToSimple SendExplicit Triggered
Interpolation Lowpass SingleDigitCounter Dim-Aktor Verschluss-Status
Verschluss-Statistik

------------------------------------------------------------------------

# Polynomial

Purpose: Universal polynomial calculation.

Syntax: \[ "Polynomial", "Input_X", "Output_Y", \[ "$A0", "$A1", ... \]
\]

Alternative: \[ "Polynomial", "Input_X", "Output_Y", \[
"\$VAR\<Koeff!\>" \] \]

Hardening Rule: Numeric literals in coefficient arrays are forbidden
(ANTI-MAGIC-NUMBERS).

------------------------------------------------------------------------

# Ratio

Purpose: Division

Syntax: \[ "Ratio", "Zaehler", "Ergebnis", "\$Nenner" \]

Critical Rule: Division by zero must be prevented.

------------------------------------------------------------------------

# Limiter

Purpose: Clamp value between min and max

Syntax: \[ "Limiter", "Input", "Out_Clamped", "Out_IsInRange", \[
"$Min", "$Max" \] \]

------------------------------------------------------------------------

# Comparator

Purpose: Threshold comparison

Syntax: \[ "Comparator", "A", "Out", "B" \]

Hysteresis: \[ "Comparator", "A", "Out", \[
"$Schwelle_Aus", "$Schwelle_Ein" \] \]

------------------------------------------------------------------------

# Logic Gates

Modules: And / Or / Xor

Syntax: \[ "Or", \[ "In_A", "In_B", ... \], "Out" \] or \[ "Or", \[
"\$VAR\<In!\>" \], "Out" \]

Inputs must be boolean.

------------------------------------------------------------------------

# Clocksignal

Syntax: \[ "Clocksignal", "Enable", "Output_Clock", "\$Perioden_Halbe_s"
\]

------------------------------------------------------------------------

# Monoflop

Syntax: \[ "Monoflop", "Set", "Reset", "Out", "Zeit_s", Mode_Int_Literal
\]

Modes: 0 level 1 rising edge 2 falling edge 3 both edges

------------------------------------------------------------------------

# Latch

Syntax: \[ "Latch", "Input", "Output", "Trigger", Mode_Int_Literal \]

------------------------------------------------------------------------

# Multiplexer

Syntax: \[ "Multiplexer", \[ "$Wert_0", "$Wert_1", ...\], "Output",
"Selektor_Int" \] or \[ "Multiplexer", \[ "\$VAR\<In!\>" \], "Output",
"Selektor_Int" \]

------------------------------------------------------------------------

# JsonPathSelector

Syntax: \[ "JsonPathSelector", "Input_Json", "JsonPath", "Output",
"Error" \]

------------------------------------------------------------------------

# HEX-\>INT

Syntax: \[ "HEX-\>INT", "Input_Hex_String", "Output_Integer",
"$ByteSwap_Bool", "$WordSwap_Bool" \]

------------------------------------------------------------------------

# STR-\>FLOAT

Syntax: \[ "STR-\>FLOAT", "Input_String", "Output_Float" \]

------------------------------------------------------------------------

# Printf

Syntax: \[ "Printf", "Input_Wert", "Format_String", "Output_String" \]

------------------------------------------------------------------------

# Regex

Syntax: \[ "Regex", "Input_Str", "Pattern_Str", "Match_Bool",
"Full_Match", "Grp1", "Grp2", "Grp3", "Grp4", "\$Grp5" \]

------------------------------------------------------------------------

# Stopwatch

Syntax: \[ "Stopwatch", "StartStop_Bool", "Time_s_Float" \]

------------------------------------------------------------------------

# PID controller

Syntax: \[ "PID controller", "Soll_float", "Ist_float", "Stell_float",
"Kp_float", "Ki_float", "Kd_float" \]

------------------------------------------------------------------------

# PID_awu

Syntax: \[ "PID_awu", "Soll_f", "Ist_f", "Stell_f", "Kp_f", "Ki_f",
"Kd_f", "Untergrenze_f", "Obergrenze_f" \]

------------------------------------------------------------------------

# Statemachine

Syntax: \[ "Statemachine", \[ \[Transition1\], \[Transition2\], ... \],
"Output_State_Int"\]

------------------------------------------------------------------------

# Statistic

Syntax: \[ "Statistic", \[ "Input1", ... \], "Out_Min", "Out_Max",
"Out_Mean", "\$Out_Median" \]

------------------------------------------------------------------------

# Ramp

Syntax: \["Ramp", "$Ziel", "$Out", "$Active", "$Step", "\$Period"\]

------------------------------------------------------------------------

# Astro

Syntax: \[ "Astro", \[ "$Latitude", "$Longitude" \], "Altitude",
"Azimute", "Transit", "Sunrise", "Sunset", "Civildawn" \]

------------------------------------------------------------------------

# Localtime

Syntax: \[ "Localtime", 0, "UnixOut", "Sec_Out", "Min_Out", "Hour_Out"
\]

------------------------------------------------------------------------

# Wakeup

Syntax: \[ "Wakeup", "UnixTimestamp_In", "Alarm_Out_Bool" \]

------------------------------------------------------------------------

# SendToSimple

Syntax: \[ "SendToSimple", "Inhibit", "Channel", "Title", "Message",
"Category", "Priority" \]

------------------------------------------------------------------------

# SendExplicit

Syntax: \[ "SendExplicit", "Sende_Bedingung_Bool", "Variable_mit_Wert",
Sende_Option_Int_Literal \]

------------------------------------------------------------------------

# Triggered

Syntax: \[ "Triggered", "Input", "Touched_Output_Bool" \]

------------------------------------------------------------------------

# Interpolation

Syntax: \[ "Interpolation", "In_X", "Out_Y", \[ \["$X1", "$Y1"\],
\["$X2", "$Y2"\] \] \]

------------------------------------------------------------------------

# Lowpass

Syntax: \[ "Lowpass", "Input", "Output", "\$Zeitkonstante_s" \]

------------------------------------------------------------------------

# SingleDigitCounter

Syntax: \["SingleDigitCounter", \["$In1", "$In2"\], "$Cnt0", "$Cnt1",
"$Cnt2", "$Cnt3", "$Cnt4", "$Cnt5", "$Cnt6", "$Cnt7", "$Cnt8", "$Cnt9"\]

------------------------------------------------------------------------

# Dim-Aktor

Syntax: \["Dim-Aktor", "$In_Relativ", "$Out_Absolut", "\$Inhibit"\]

------------------------------------------------------------------------

# Verschluss-Status

Syntax: \["Verschluss-Status", "$Kipp", "$Offen",
"$StatusInt", "$OffenBool", "\$Text"\]

------------------------------------------------------------------------

# Verschluss-Statistik

Syntax: \["Verschluss-Statistik", \["$StatusInt1"], "$AnzahlOffen",
"$AnzahlGekippt", "$AllesZu"\]
