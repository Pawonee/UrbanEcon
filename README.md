# UrbanEcon
The Efficiency of Race-Neutral Alternatives to Race-Based Affirmative Action: Evidence from Chicago's Exam Schools
Replication attempt by - Pawonee Khadka

The original paper is by Ellison and Pathak (2020). 
Refer to attached link for AER : https://www.openicpsr.org/openicpsr/project/120529/version/V1/view?path=/openicpsr/120529/fcr:versions/V1&type=project

Please note that this was the initial project i began work with. Unfortunately, after some time I discovered data was missing and I couldn't do any analysis. Following is a quick summary of my initial work:



I've already installed some packages as per instruction:
install.packages(c("leaps","gelnet","doParallel","foreach","hdm"))
I have attached here a snap of the first set of instructions for cleaning tracts data using stata. I will refer to it as I do so in R. The resulting work will be in a markdwn file.

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


I cleaned the above data in R and created a quick anaylsis.The authors have a lot of the raw data, so I am able to do these quick cleans in multiple stages, however, there is key missing data which restricted me from replication.
