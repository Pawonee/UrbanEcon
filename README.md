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

*~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
*   Tabulates efficiency ratios for each of the alternative race-nuetral
*	affirmative action plans
*~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
*Specifications we want to tabulate
local Payton_allocations tier_.175 ///
tract_ses2_.114 ///
tract_ses3_.089 tract_ses2_lunch_.106 ///
tract_ses3_lunch_.074 match_nar${ref_benchm_pay}_allocations ///
chgoca_score_.31 tract_score_.295 tract_ses5_.144 tract_ses4_.1155

local Northside_allocations  tier_.175 ///
tract_ses2_.145 ///
tract_ses3_.111 tract_ses2_lunch_.132 ///
tract_ses3_lunch_.101 match_nar${ref_benchm_nor}_allocations ///
chgoca_score_.34 tract_score_.37 tract_ses5_.183 tract_ses4_.1455

****** I) Get reference Values *******

*This series of do files identifies benchmark plans that get different
*minority frpl shares
do "${code}setup/allocation_search_ovrl.do"
do "${code}setup/allocation_search_frpl.do"
do "${code}setup/allocation_search_mino.do"

import delimited "${rawdata}applications/SEHS2011_Applications.csv" , clear

rename _kf_applicationid appid
keep if !mi(appid)

gen points_total_selective = cho_app_applicationpointstotal_s/9

keep if inlist(schoolname,"Payton","Northside")
keep if inlist(selectionstatus,"==selected","Selected")

gen offer2011_payton = schoolname == "Payton"
gen offer2011_northside = schoolname == "Northside"

gen enrol2011_payton 	= schoolname == "Payton" & selectionresponse == "Accepted"
gen enrol2011_northside = schoolname == "Northside" & selectionresponse == "Accepted"

*Look at points if someone received an offer at each point in time
foreach var of varlist offer2011* enrol2011* {
	sum points_total_selective if `var' == 1
	global meanp_`var' = r(mean)
}

*Track some additional variables to tabulate
preserve
	use "${data}figures", clear
	egen student = group(appid)
	gen asian = (ethnicity == "Asian")
	keep student appid asian pts_exam pts_grades pts_seven
	duplicates drop
	//rescale to be out of 100
	foreach pts of varlist pts*{
		replace `pts' = `pts'/3
	}
	tempfile additionalvars
	save "`additionalvars'"
restore


foreach School in Payton Northside {

	if "`School'" == "Payton" {
		local school payton
		local school_short pay
	}

	if "`School'" == "Northside" {
		local school  northside
		local school_short  nor
	}

	local nr = 0

	foreach allocation of local `School'_allocations {

		if "`allocation'" == "tier_.175" 		{
			local allocation_str = "tier"
			local ref_value = 5
		}

		if "`allocation'" == "tract_ses2_.114" | "`allocation'" == "tract_ses2_.145" 		{
			local allocation_str = "rewei1"
			local ref_value = 1
		}
		if "`allocation'" == "tract_ses2_lunch_.106" | "`allocation'" == "tract_ses2_lunch_.132"	{
			local allocation_str = "rewei2"
			local ref_value = 2
		}
		if "`allocation'" == "tract_ses3_.089" | "`allocation'" == "tract_ses3_.111" 	{
			local allocation_str = "lasso1"
			local ref_value = 3
		}
		if "`allocation'" == "tract_ses3_lunch_.074" | "`allocation'" == "tract_ses3_lunch_.101"		{
			local allocation_str = "lasso2"
			local ref_value = 4
		}
		if "`allocation'" == "match_nar${ref_benchm_`school_short'}_allocations"	{
			local allocation_str = "benchm"
			local ref_value = 5
		}

		if "`allocation'" == "chgoca_score_.31" | "`allocation'" == "chgoca_score_.34" 	{
			local allocation_str = "t10com"
			local ref_value = 5
		}
		if "`allocation'" == "tract_score_.295" | "`allocation'" == "tract_score_.37" 	{
			local allocation_str = "t10tra"
			local ref_value = 5
		}
		if "`allocation'" == "tract_ses5_.144" | "`allocation'" == "tract_ses5_.183" 	{
			local allocation_str = "unweig"
			local ref_value = 5
		}
		if "`allocation'" == "tract_ses4_.1455" | "`allocation'" == "tract_ses4_.1155" 	{
			local allocation_str = "unweig2"
			local ref_value = 5
		}

		// Step 1: Get Benchmark Stats
		*The steps for each block below are
		*    1. Load in the relevant allocation.
		*    2. Merge in student information
		*    3. Calculate the distributional information for that allocation
		// Overall
		use "${data}/allocations/match_nar${ref_`allocation_str'_`school_short'}_allocations.dta", clear
		rename points_total_selective pts
		merge m:1 student using "`additionalvars'", keepusing(pts*) keep(1 3)
		foreach p of varlist pts*{
			sum `p' if school == "`School'"
			scalar var_`p'_bench_`allocation_str' = r(Var)
			scalar sd_`p'_bench_`allocation_str' = r(sd)
			scalar mean_`p'_bench_`allocation_str' = r(mean)
		}
		tokenize "${ref_`allocation_str'_`school_short'}", parse("_")
		scalar mpoints_bench_`allocation_str' = 1.5 + .05*`1'
		scalar fpoints_bench_`allocation_str' = 0.5 + .05*`3'

		// Minority
		use "${data}/allocations/match_reg${ref_mino_`school_short'}_allocations.dta", clear
		rename points_total_selective pts
		merge m:1 student using "`additionalvars'", keepusing(pts*) keep(1 3)
		foreach p of varlist pts*{
			sum `p' if school == "`School'"
			scalar var_`p'_mbench_`allocation_str' = r(Var)
			scalar sd_`p'_mbench_`allocation_str' = r(sd)
			scalar mean_`p'_mbench_`allocation_str' = r(mean)
		}
		tokenize "${ref_mino_`school_short'}", parse("_")
		scalar mpoints_mbench_`allocation_str' = .1*`1'
		scalar fpoints_mbench_`allocation_str' = .1*`3'

		// FRPL
		use "${data}/allocations/match_reg${ref_frpl_`allocation_str'_`school_short'}_allocations.dta", clear
		rename points_total_selective pts
		merge m:1 student using "`additionalvars'", keepusing(pts*) keep(1 3)
		foreach p of varlist pts*{
			sum `p' if school == "`School'"
			scalar var_`p'_fbench_`allocation_str' = r(Var)
			scalar sd_`p'_fbench_`allocation_str' = r(sd)
			scalar mean_`p'_fbench_`allocation_str' = r(mean)
		}
		tokenize "${ref_frpl_`allocation_str'_`school_short'}", parse("_")
		scalar mpoints_fbench_`allocation_str' = .1*`1'
		scalar fpoints_fbench_`allocation_str' = .1*`3'

		// Points
		use "${data}/allocations/points.dta", clear
		rename points_total_selective pts
		merge m:1 appid using "`additionalvars'", keepusing(pts*) keep(1 3)
		foreach p of varlist pts*{
			sum `p' if school == "`School'"
			scalar var_`p'_score_based = r(Var)
			scalar sd_`p'_score_based = r(sd)
			scalar mean_`p'_score_based = r(mean)
		}

		*now load in the information calculated above to measure efficiencies and different
		*shares.
		use "${data}/allocations/`allocation'.dta", clear
		rename points_total_selective pts
		if "`allocation_str'" == "benchm" merge m:1 student using "`additionalvars'", keep(1 3)
		else merge m:1 appid using "`additionalvars'", keep(1 3)

		foreach p of varlist pts*{
			sum `p' if school == "`School'"

			// Composite Score Mean and Variance
			gen mean_`p'_`allocation_str' = r(mean)
			gen var_`p'_`allocation_str' = r(Var)
			gen sd_`p'_`allocation_str' = r(sd)

			// Efficiency Measures
			gen oveff_`p'_`allocation_str' = (var_`p'_bench_`allocation_str' - var_`p'_score_based) / (var_`p'_`allocation_str' - var_`p'_score_based)
			gen mineff_`p'_`allocation_str' = (var_`p'_mbench_`allocation_str' - var_`p'_score_based) / (var_`p'_`allocation_str' - var_`p'_score_based)
			gen luneff_`p'_`allocation_str' = (var_`p'_fbench_`allocation_str' - var_`p'_score_based) / (var_`p'_`allocation_str' - var_`p'_score_based)

			gen ovsdeff_`p'_`allocation_str' = (sd_`p'_bench_`allocation_str' - sd_`p'_score_based) / (sd_`p'_`allocation_str' - sd_`p'_score_based)
			gen minsdeff_`p'_`allocation_str' = (sd_`p'_mbench_`allocation_str' - sd_`p'_score_based) / (sd_`p'_`allocation_str' - sd_`p'_score_based)
			gen lunsdeff_`p'_`allocation_str' = (sd_`p'_fbench_`allocation_str' - sd_`p'_score_based) / (sd_`p'_`allocation_str' - sd_`p'_score_based)

			gen ovmeaneff_`p'_`allocation_str' = (mean_`p'_bench_`allocation_str' - mean_`p'_score_based) / (mean_`p'_`allocation_str' - mean_`p'_score_based)
			gen minmeaneff_`p'_`allocation_str' = (mean_`p'_mbench_`allocation_str' - mean_`p'_score_based) / (mean_`p'_`allocation_str' - mean_`p'_score_based)
			gen lunmeaneff_`p'_`allocation_str' = (mean_`p'_fbench_`allocation_str' - mean_`p'_score_based) / (mean_`p'_`allocation_str' - mean_`p'_score_based)

			//Minority Gap
			reg `p' minority if school == "`School'"
			matrix mingap = e(b)
			scalar mingap_`p'_`allocation_str' = -mingap[1,1]
			gen mingap_`p'_`allocation_str' = mingap_`p'_`allocation_str'
		}

		// Minority Share
		sum minority if school == "`School'"
		gen minshare_`allocation_str' = r(mean)

		// Asian Share
		sum asian if school == "`School'"
		gen asianshare_`allocation_str' = r(mean)

		// FRPL Share
		sum frpl if school == "`School'"
		gen lunshare_`allocation_str' = r(mean)

		// Bonus points
		gen mpoints_bench_`allocation_str' = mpoints_bench_`allocation_str'
		gen fpoints_bench_`allocation_str' = fpoints_bench_`allocation_str'
		gen mpoints_mbench_`allocation_str' = mpoints_mbench_`allocation_str'
		gen fpoints_mbench_`allocation_str' = fpoints_mbench_`allocation_str'
		gen mpoints_fbench_`allocation_str' = mpoints_fbench_`allocation_str'
		gen fpoints_fbench_`allocation_str' = fpoints_fbench_`allocation_str'

		keep if _n == 1
		keep *_`allocation_str'
		rename *_`allocation_str' *

		*Formatting for the table
		gen spec = "`School'_`allocation_str'"
		local cols spec minshare lunshare asianshare
		foreach p in pts pts_exam pts_seven pts_grades{
			local cols `cols' mean_`p' sd_`p' mingap_`p' ovsdeff_`p' minsdeff_`p' lunsdeff_`p' ovmeaneff_`p' minmeaneff_`p' lunmeaneff_`p'
		}
		local cols `cols' mpoints_bench fpoints_bench mpoints_mbench fpoints_mbench mpoints_fbench fpoints_fbench
		order `cols'
		keep `cols'

		if "`allocation_str'" == "tier" {
			tempfile table_3_`school'
			save `table_3_`school''
		}

		else {

			tempfile table_3_`allocation_str'
			save `table_3_`allocation_str''

			use `table_3_`school'', clear
			append using `table_3_`allocation_str''
			save `table_3_`school'', replace

		}
	}
}

use `table_3_payton'
append using `table_3_northside'

outsheet using "${tables}table3_A7.csv", c replace

