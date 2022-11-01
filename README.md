# UrbanEcon
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

insheet using "${rawdata}nhgis0005_csv/nhgis0005_ds206_20145_2014_tract.csv", clear
keep if statea == 17 & countya == 31
drop ????m???
gen tract = string(tract)
merge 1:1 tract using `cps', assert(1 3) keep(3) nogen
save ${data}tables/table1.dta, replace

insheet using "${rawdata}nhgis0005_csv/nhgis0005_ds207_20145_2014_tract.csv", clear
keep if statea == 17 & countya == 31
drop ????m???
gen tract = string(tract)
merge 1:1 tract using `cps', assert(1 3) keep(3) nogen
save ${data}tables/table2.dta, replace

* Use Glenn's definitions
use ${data}tables/table1.dta, clear
merge 1:1 tract using ${data}tables/table2.dta, assert(3) nogen
ren *, upper
gen	v1 = (ABAQE012+ABAQE013)/(ABAQE012+ABAQE013+ABAQE036+ABAQE037)
gen	v2 = (ABAQE003+ABAQE004+ABAQE027+ABAQE028)/ABAQE001
gen	v3 = (ABAQE024+ABAQE025+ABAQE048+ABAQE049)/ABAQE001
gen	v4 = ABARE001
gen	v5 = ABARE002
gen	v6 = ABBCE003/ABBCE001
gen	v7 = ABBCE014/ABBCE001
gen	v8 = ABBRE002/ABBRE001
gen	v9 = ABBRE011/ABBRE001
gen	v10 = ABBRE013/ABBRE001
gen	v11 = ABBRE019/ABBRE001
gen	v12 = (ABBSE002 + ABBSE003 + ABBSE004)/ABBSE001
gen	v13 = (ABBSE011 + ABBSE012)/ABBSE001
gen	v14 = (ABBSE014 + ABBSE015)/ABBSE001
gen	v15 = (ABBTE012 + ABBTE013)/ABBTE001
gen	v16 = (ABBTE010 + ABBTE011)/ABBTE001
gen	v17 = ABBUE002/ABBUE001
gen	v18 = ABBUE004/(ABBUE004+ABBUE011+ABBUE017)
gen	v19 = ABBUE006/(ABBUE006+ABBUE013+ABBUE019)
gen	v20 = ABBUE007/(ABBUE007+ABBUE014+ABBUE020)
gen	v21 = ABBVE006/ABBVE001
gen	v22 = ABBVE008/ABBVE001
gen	v23 = ABBYE007/ABBYE001
gen	v24 = ABBYE006/ABBYE002
gen	v25 = ABCIE003/(ABCIE003+ABCIE010+ABCIE016)
gen	v26 = ABCIE005/(ABCIE005+ABCIE012+ABCIE018)
gen	v27 = ABCIE006/(ABCIE006+ABCIE013+ABCIE019)
gen	v28 = ABCOE002/ABCOE001
gen	v29 = (ABC4E002+ABC4E003+ABC4E004+ABC4E005+ABC4E006+ABC4E007+ABC4E008+ABC4E009+ABC4E010+ABC4E011+ABC4E012)/ABC4E001
gen	v30 = (ABC4E013+ABC4E014+ABC4E015+ABC4E016)/ABC4E001
gen	v31 = ABC4E017/ABC4E001
gen	v32 = ABC4E018/ABC4E001
gen	v33 = (ABC4E019+ABC4E020)/ABC4E001
gen	v34 = ABC4E011/ABC4E001
gen	v35 = ABC4E022/ABC4E001
gen	v36 = (ABC4E023+ABC4E024)/ABC4E001
gen	v37 = ABC4E025/ABC4E001
gen percent_lt_hs = (ABC4E002+ABC4E003+ABC4E004+ABC4E005+ABC4E006+ABC4E007+ABC4E008+ABC4E009+ABC4E010+ABC4E011+ABC4E012+ABC4E013+ABC4E014+ABC4E015+ABC4E016)/ABC4E001
gen percent_hs = (ABC4E017+ABC4E018)/ABC4E001
gen percent_sc = (ABC4E019+ABC4E020+ABC4E021)/ABC4E001
gen percent_coll = ABC4E022/ABC4E001
gen percent_adv = (ABC4E023+ABC4E024+ABC4E025)/ABC4E001
gen v38 = .2*percent_lt_hs+.4*percent_hs+.6*percent_sc+.8*percent_coll+percent_adv
drop percent*
*gen	v38 = Row30*0.2+Row31*0.2+Row32*0.4+Row33*0.4+Row34*0.6+Row35*0.6+Row36*0.8+Row37*1+Row38*1
gen	v39 = ABC5E002/ABC5E001
gen	v40 = ABC5E004/ABC5E001
gen	v41 = ABC5E005/ABC5E001
gen	v42 = (ABDGE002+ABDGE003+ABDGE004+ABDGE007)/ABDGE001
gen	v43 = (ABDHE004+ABDHE007+ABDHE010+ABDHE013)/ABDHE001
gen	v44 = (ABDIE006+ABDIE011+ABDIE016+ABDIE021)/ABDIE002
gen	v45 = (ABDIE007+ABDIE012+ABDIE017+ABDIE022)/ABDIE002
gen	v46 = (ABDIE008+ABDIE013+ABDIE018+ABDIE023)/ABDIE002
gen	v47 = (ABDIE029+ABDIE034+ABDIE039+ABDIE044)/ABDIE024
gen	v48 = (ABDIE030+ABDIE035+ABDIE040+ABDIE045)/ABDIE024
gen	v49 = (ABDJE002+ABDJE003)/ABDJE001
gen	v50 = (ABDJE004+ABDJE005)/ABDJE001
gen	v51 = (ABDJE006+ABDJE007)/ABDJE001
gen	v52 = ABDJE008/ABDJE001
gen	v53 = ABDOE017/ABDOE001
gen	v54 = (ABDOE014+ABDOE015+ABDOE016)/ABDOE001
gen	v55 = (ABDOE011+ABDOE012+ABDOE013)/ABDOE001
gen	v56 = log(ABDPE001)
gen	v57 = ABECE003/ABECE001
gen	v58 = ABEDE003/ABEDE001
gen	v59 = ABEEE003/ABEEE001
gen	v60 = ABEFE003/ABEFE001
gen	v61 = ABEGE003/ABEGE001
gen	v62 = ABEHE002/ABEHE001
gen	v63 = ABEIE002/ABEIE001
gen	v64 = log(ABFIE001)
gen	v65 = ABF5E002/ABF5E001
gen	v66 = ABF5E026/ABF5E025
gen	v67 = ABF5E028/(ABF5E028+ABF5E010)
gen	v68 = ABF8E002/ABF8E001
gen	v69 = ABGFE007/ABGFE001
gen	v70 = ABGFE005/(ABGFE004+ABGFE005)
gen	v71 = ABGHE011/ABGHE007
gen	v72 = ABGHE016/ABGHE012
gen	v73 = ABGHE021/ABGHE017
gen	v74 = ABGHE026/ABGHE022
gen	v75 = ABGUE002/ABGUE001
gen	v76 = (ABGUE003+ABGUE013)/ABGUE001
gen	v77 = ABGXE003/ABGXE001
gen	v78 = ABGXE002/ABGXE001
gen	v79 = ABG7E002/ABG7E001
gen	v80 = ABG7E008/ABG7E001
gen	v81 = ABHHE001
gen	v82 = ABHIE001/ABHGE001
gen	v83 = ABHME002/ABHME001
gen	v84 = (ABHME002+ABHME003)/ABHME001
gen	v85 = (ABHME004+ABHME005)/ABHME001
gen	v86 = ABHME009/ABHME001
gen	v87 = ABHPE010/ABHPE001
gen	v88 = (ABHPE007+ABHPE008+ABHPE009)/ABHPE001
gen	v89 = (ABHPE002+ABHPE003)/ABHPE001
gen	v90 = 2016-ABHQE001
gen	v91 = ABHWE002/ABHWE001
gen	v92 = (ABHWE002+ABHWE003)/ABHWE001
gen	v93 = (ABHWE005+ABHWE006+ABHWE007)/ABHWE001
gen	v94 = (ABHWE006+ABHWE007)/ABHWE001
gen	v95 = (ABHZE003+ABHZE010)/ABHZE001
gen	v96 = (ABHZE003+ABHZE010+ABHZE004+ABHZE011)/ABHZE001
gen	v97 = log(ABIAE001)
gen	v98 = log(ABIBE001)
gen	v99 = log(ABICE001)
gen	v100 = log(ABISE001)
gen	v101 = log(ABITE001)
gen	v102 = log(ABIUE001)
gen	v103 = (ABIXE003+ABIXE006)/ABIXE002
gen	v104 = ABLIE006/ABLIE001
gen	v105 = ABLIE002/ABLIE001
gen	v106 = ABLYE002/ABLYE001
gen	v107 = ABLYE047/ABLYE001
gen	v108 = (ABL4E004+ABL4E005)/ABL4E003
gen	v109 = ABM9E019/ABM9E003
gen	v110 = ABM9E083/ABM9E003
gen	v111 = ABOHE001/ABBSE001
gen	v112 = ABOIE002/ABOIE001
gen	v113 = ABPDE006/ABPDE001
gen	v114 = ABPDE005/ABPDE002
gen	v115 = ABPEE011/(ABPEE011+ABPEE005)
gen	v116 = ABQ2E009/(ABQ2E009+ABQ2E018)
gen	v117 = ABQ2E009/(ABQ2E009+ABQ2E005)
gen	v118 = ABRIE003/(ABRIE003+ABRIE011)
gen	v119 = ABTBE003/(ABTBE003+ABTBE011)
gen	v120 = ABTBE004/(ABTBE004+ABTBE012)
gen	v121 = ABTBE005/(ABTBE005+ABTBE013)
gen	v122 = ABTQE013/ABTQE012
gen	v123 = (ABTQE004+ABTQE023)/(ABTQE003+ABTQE022)
gen	v124 = (ABTQE007+ABTQE026)/(ABTQE006+ABTQE025)
gen	v125 = ABUWE002/ABUWE001
gen	v126 = log(ABUXE001)
gen	v127 = log(ABUXE002)
gen	v128 = log(ABUXE003)
gen	v129 = log(ABUXE004)
gen	v130 = log(ABUXE005)
gen	v131 = log(ABUYE001)
gen	v132 = log(ABUYE002)
gen	v133 = log(ABUYE003)
gen	v134 = log(ABUYE004)
gen	v135 = log(ABUYE005)
gen	v136 = log(ABUYE006)
gen	v137 = ABU0E001
gen	v138 = ABWGE002/ABWGE001
gen	v139 = ABWLE003/(ABWLE003+ABWLE016)
gen	v140 = ABWLE007/ABWLE001
gen	v141 = log(ABY2E001)
gen	v142 = ABZRE001/ABLIE001
gen	v143 = (ABZSE004+ABZSE032)/(ABZSE003+ABZSE031)
gen	v144 = (ABZSE007+ABZSE035)/(ABZSE006+ABZSE034)
gen	v145 = (ABZSE013+ABZSE041+ABZSE016+ABZSE044)/(ABZSE012+ABZSE040+ABZSE015+ABZSE043)

label var v1 "Fraction of 30-39 year olds who are male"
label var v2 "Fraction of population aged 9 and under"
label var v3 "Fraction of population aged 80 and older"
label var v4 "Median age"
label var v5 "Median age of males"
label var v6 "Fraction of people who moved within US in last year"
label var v7 "Fraction living abroad 1 year ago"
label var v8 "Fraction of workers driving to work"
label var v9 "Fraction commuting by bus"
label var v10 "Fraction commuting by subway or elevated train"
label var v11 "Fraction walking to work"
label var v12 "Fraction leaving for work 12am-6am"
label var v13 "Fraction leaving for work 9am-11am"
label var v14 "Fraction leaving for work 12pm-12am"
label var v15 "Fraction commuting 60+ minutes"
label var v16 "Fraction commuting 40-59 minutes"
label var v17 "Fraction of children in married couple households"
label var v18 "Fraction of childred 3-4 in married couple households"
label var v19 "Fraction of childred 6-11 in married couple households"
label var v20 "Fraction of childred 12-17 in married couple households"
label var v21 "Fraction of children who are grandchild of head of household"
label var v22 "Fraction of children who are foster child or unrelated to head of household"
label var v23 "Fraction of households that are nonfamily"
label var v24 "Fraction of family household with no husband present"
label var v25 "Fraction of family households with children under 18 with married couple"
label var v26 "Fraction of family households with children under 6 and 6-17  with married couple"
label var v27 "Fraction of family households with children 6-17 only with married couple"
label var v28 "Fraction of households with a nonrelative"
label var v29 "Fraction of 25+ with at most 8th grade education"
label var v30 "Fraction of 25+ with some HS but no diploma"
label var v31 "Fraction of 25+ with HS diploma"
label var v32 "Fraction of 25+ with GED"
label var v33 "Fraction of 25+ with some college no degree"
label var v34 "Fraction of 25+ with Associates"
label var v35 "Fraction of 25+ with Bachelors"
label var v36 "Fraction of 25+ with Master's or Professional"
label var v37 "Fraction of 25+ with Doctoral degree"
label var v38 "Aggregation of adult education"
label var v39 "Fraction of Bachelors with Sci and Eng Degrees"
label var v40 "Fraction of Bachelor's with Business degrees"
label var v41 "Fraction of Bachelor's with Education degrees"
label var v42 "Fraction of Bachelor's with STEM degrees"
label var v43 "Fraction of households limited English speaking"
label var v44 "Fraction of 5-17 year olds speaking English well but not very well"
label var v45 "Fraction of 5-17 year olds speaking English not well"
label var v46 "Fraction of 5-17 year olds speaking English not at all"
label var v47 "Fraction of 18-64 year olds speaking English not well"
label var v48 "Fraction of 18-64 year olds speaking English not at all"
label var v49 "Fraction income below poverty"
label var v50 "Fraction income 1.0-1.49 poverty"
label var v51 "Fraction income 1.5-1.99 poverty"
label var v52 "Fraction income 2 x poverty and up"
label var v53 "Fraction household income >=$200,000"
label var v54 "Fraction household income [$100,000,$200,000)"
label var v55 "Fraction household income [$50,000,$100,000)"
label var v56 "Median household income"
label var v57 "Fraction of households with no earnings"
label var v58 "Fraction of households with no wage or salary income"
label var v59 "Fraction of households with no self-employment income"
label var v60 "Fraction of households with no interest, dividend, or net rental income"
label var v61 "Fraction of households with no social security income"
label var v62 "Fraction of households with SSI income"
label var v63 "Fraction of households with public assistance income"
label var v64 "Per capita income"
label var v65 "Fraction Civilian 18+ who are veterans"
label var v66 "Fraction Civilian Females 18-34 who are veterans"
label var v67 "Fraction of 35-54 Civilians Who are Female"
label var v68 "Fraction of households with Food Stamps/SNAP in last year"
label var v69 "Fraction of 16+ not in labor force"
label var v70 "Fraction of 16+ civilian labor force who are unemployed"
label var v71 "Fraction of 20-24 year olds who did not work in past 12 months"
label var v72 "Fraction of 25-44 year olds who did not work in past 12 months"
label var v73 "Fraction of 45-54 year olds who did not work in past 12 months"
label var v74 "Fraction of 55-64 year olds who did not work in past 12 months"
label var v75 "Fraction of 16+ civilian employed workers who are male"
label var v76 "Fraction of 16+ civilian employed who are private for-profit workers"
label var v77 "Fraction of housing units that are vacant"
label var v78 "Fraction of occupied housing units owner-occupied"
label var v79 "Fraction of vacant units for rent"
label var v80 "Fraction of vacant units other vacant"
label var v81 "Median number of rooms per housing unit"
label var v82 "Mean number of rooms per housing unit"
label var v83 "Fraction of single-family detached homes"
label var v84 "Fraction of single-family detached or attached homes"
label var v85 "Fraction of housing units in 2-4 family structures"
label var v86 "Fraction of housing units in 50+ unit structures"
label var v87 "Fraction of housing units pre-1940"
label var v88 "Fraction of housing units built 1940-1969"
label var v89 "Fraction of housing units built 2000 or later"
label var v90 "Median age for housing units"
label var v91 "Fraction of housing units with no BRs"
label var v92 "Fraction of housing units with 0-1 BR"
label var v93 "Fraction of housing units with 3 or more BR"
label var v94 "Fraction of housing units with 4 or more BR"
label var v95 "Fraction of occupied units with no vehicle"
label var v96 "Fraction of occupied units with fewer than 2 vehicles"
label var v97 "Lower quartile contract rent for renter occupied paying rent"
label var v98 "Median contract rent for renter occupied paying rent"
label var v99 "Upper quartile contract rent for renter occupied paying rent"
label var v100 "Lower value quartile for owner-occupied units"
label var v101 "Median value for owner occupied units"
label var v102 "Upper value quartile for owner occupied units"
label var v103 "Fraction of mortgaged units with second mortgate or home equity loan"
label var v104 "Fraction not US citizens"
label var v105 "Fraction who are US citizens born in US"
label var v106 "Fraction of foreign born from Europe"
label var v107 "Fraction of foreign born from Asia"
label var v108 "Fraction of naturalized citizens naturalized 2005 or later"
label var v109 "Fraction of 5-17 year olds in same house as 1 year ago"
label var v110 "Fraction of 5-17 year olds not in US 1 year ago"
label var v111 "Average commuting time for workers 16+ not working at home"
label var v112 "Fraction workers 16+ with no vehicle available"
label var v113 "Fraction of children under 18 in nonfamily households"
label var v114 "Fraction of children in family households for which no husband present"
label var v115 "Fraction of children in no husband families who also have no unmarried partner present"
label var v116 "Fraction of unmarried 20-34 year old women who had a child in past 12 months"
label var v117 "Fraction now married among 20-34 year olds who gave birth in last 12 months"
label var v118 "Fraction of 15-19 year old women who had a child in past 12 months"
label var v119 "Fraction of children under 6 with income below poverty"
label var v120 "Fraction of 6-11 years olds with income below poverty"
label var v121 "Fraction of 12-17 years olds with income below poverty"
label var v122 "Fraction of males 35-64 with disability"
label var v123 "Fraction of children under 5 with disability"
label var v124 "Fraction of children 5-17 with disability"
label var v125 "Fraction of households with cash public assistance or Food Stamps/SNAP"
label var v126 "HH Income 20th percentile"
label var v127 "HH Income 40th percentile"
label var v128 "HH Income 60th percentile"
label var v129 "HH Income 80th percentile"
label var v130 "HH Income 95th percentile"
label var v131 "Mean HH income within bottom quintile"
label var v132 "Mean HH income within 2nd lowest quintile"
label var v133 "Mean HH income within middle quintile"
label var v134 "Mean HH income within 2nd highest quintile"
label var v135 "Mean HH income within top quintile"
label var v136 "Mean HH income within top 5 percent"
label var v137 "Gini index for HH income"
label var v138 "Fraction veterans among civilian 25+ population"
label var v139 "Fraction of HH with children under 18 who received Food Stamps/SNAP"
label var v140 "Fraction of HH that are female head, no husband, and Food Stampss/SNAP"
label var v141 "Median monthly housing costs for occupied units with costs"
label var v142 "Fraction of US population in group quarterns"
label var v143 "Fraction of under 6 year olds with health insurance"
label var v144 "Fraction of 6-17 year olds with health insurance"
label var v145 "Fraction of 25-44 year olds with health insurance"

ren *, lower
ren weightedaverageisatperformanceat isat
keep tract v*
order tract v*
save "${data}census_vars.dta", replace
