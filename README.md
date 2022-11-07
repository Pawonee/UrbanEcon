# UrbanEcon


Refer to attached link for AER : https://www.openicpsr.org/openicpsr/project/120529/version/V1/view?path=/openicpsr/120529/fcr:versions/V1&type=project

I've already installed some packages as per instruction:
install.packages(c("leaps","gelnet","doParallel","foreach","hdm"))

Following is the first set of instructions for cleaning tracts data using stata. I will refer to it as I do so in R.

* CPS tract characteristics data
insheet using "${rawdata}Summary of Census Socioeconomic Data 2012-2013.csv", clear
destring *, replace ignore("%")
ren estimatedmedianfamilyincomecumul income
ren educationalattainmentscorecumula educ
ren ofsingleparenthouseholdscumulati single
ren ofowneroccupiedhomescumulativepe owner
ren v18 english
ren v19 isat
ren tract tracta
replace tracta = 100*tracta
gen tract = string(tracta)
foreach v of varlist income educ single owner english isat {
  replace `v' = .01*`v'
}
keep tract tracta income educ single owner english isat factorsocioeconomictier
save "${data}tract_char.dta", replace

insheet using "${rawdata}/Summary of Census Socioeconomic Data 2012-2013.csv", clear
keep tract *isat*
ren tract tracta
gen tract = string(100*tract)
tempfile cps
save `cps'


First stage of the project is completed with data cleaning. The next step is to refer to the already cleaned data and start working on replication:

