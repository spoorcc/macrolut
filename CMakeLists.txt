cmake_minimum_required(VERSION 3.0)
project(MacroLUT)

add_executable(lut_test lut_test.c)

macro(gnuplot_lut function xrange samples filename)

file(WRITE ${filename}
"#!/usr/bin/env gnuplot
set print \"-\"
set samples 11
set xrange ${xrange}
f(x) = ${function}
set table $myData
plot f(x) notitle
unset table
print $myData"
)

endmacro(gnuplot_lut)

macro(create_lut function xrange samples)
    set(GNUPLOTFILE ${CMAKE_BINARY_DIR}/generated_lut.gnuplot)
    message("-- Created gnuplot file ${GNUPLOTFILE} for ${function}")
    gnuplot_lut(${function} ${xrange} ${samples} ${GNUPLOTFILE})
    execute_process(COMMAND gnuplot ${GNUPLOTFILE}
                    COMMAND tail -n+4
                    COMMAND cut -f 4 -d " "
                    COMMAND tr '\n' ','
                    OUTPUT_FILE lutvalues.h)
endmacro(create_lut)

create_lut(x**2 [0:10] 10)
