# Flypaper
* Flypaper in Educ*
/*
Coletando as  informações do SPAECE alfa disponiveis em:
https://www.seduc.ce.gov.br/resultado-spaece-alfa/ ,
bem como os microdados disponiblizados do anos de 20007 a 2015, disponivel em:
http://inep.gov.br/microdados & http://portal.inep.gov.br/indicadores-educacionais

*/

********************************************************************************
*******             Organização Base ESCOLA PROFICÊNCIA             ************
********************************************************************************
clear all

*Base inicial que será usada no Formato bruto. 
use "D:\Pedro\Tese Pedro\Flypaper\Pareamento\censo\spaece_inicial.dta"



keep if ano== 2007
gen d_07 = 1

keep cod_escola d_07

merge m:m cod_escola using "D:\Pedro\Tese Pedro\Flypaper\Pareamento\censo\spaece_inicial.dta"
replace d_07 =0 if d_07 ==.

move cod_escola mun
move d_07 tdi_anos_iniciais



rename cod_mun cod_ibge

drop _merge
 
label variable d_07 "Escola que aparece em 2007 ao longo dos anos"

save "D:\Pedro\Tese Pedro\Flypaper\Pareamento\censo\spaece_inicial_1.dta", replace


egen rank = xtile(prof_med), by(cod_ibge ano) nq(4)

label variable rank "rank das escolas em cada ano"

save "D:\Pedro\Tese Pedro\Flypaper\Pareamento\censo\spaece_inicial_1.dta", replace

bysort ano: tab d_07 rank

keep if ano ==2007 

keep cod_escola rank

rename rank rank_07

tab rank_07

merge m:m cod_escola using "D:\Pedro\Tese Pedro\Flypaper\Pareamento\censo\spaece_inicial_1.dta"

move rank_07 rank
move cod_escola mun

label variable rank_07 "rank das escolas em 2007"

drop _merge



gen tratamento =.
replace tratamento =1 if rank_07 ==1
replace tratamento =0 if rank_07 ==2
replace tratamento =0 if rank_07 ==3
replace tratamento =0 if rank_07 ==4

tab tratamento ano, m

*Abaibara e Altaneira com menos de 4 escolas em 2007

drop if cod_ibge== 2300101
drop if cod_ibge== 2300606

label variable tratamento "1º quartil de 2007 ao longo dos anos"


gen it= ano*cod_escola
label variable it "interação ano* escola"

egen std_prof = std(prof_med)
move std_prof prof_med


areg std_prof i.ano tratamento , a(it) 


save "D:\Pedro\Tese Pedro\Flypaper\Pareamento\censo\spaece_inicial_1.dta", replace


********************************************************************************
*******             Organização da Base COTA PARTE                  ************
********************************************************************************

* Base Inicial é a Base no formato Bruto. Dela serão obtidas as demais bases.
clear all



use "D:\Pedro\Tese Pedro\Flypaper\flypaper-rafa\Base Inicial.dta"
drop if ano==2018
merge m:m ano using "D:\Pedro\Tese Pedro\Flypaper\flypaper-rafa\inf_dessa.dta"
gen id=_n
drop _merge
merge m:m id using "D:\Pedro\Tese Pedro\Flypaper\flypaper-rafa\controls.dta"
drop _merge
drop in 2577


format %13.0g TransferênciasdoFUNDEB
format %13.0g cotapartefpm

merge m:m id using "D:\Pedro\Tese Pedro\Flypaper\flypaper-rafa\valor_real_cp.dta"
drop _merge	

save "D:\Pedro\Tese Pedro\Flypaper\flypaper-rafa\Base Inicial 2.dta", replace

* Construção das variáveis de resultado e controle iniciais
***********************************************************

* As variáveis de resultado serão:
* 1. Gasto Total Per Capita Relativo a 2008
* 2. Gasto em Educação Total Per Capita Relativo a 2008
* 3. Gasto em Educação Fundamental Per capita relativo a 2008.

******
* Seja G(t) uma variável de gasto no tempo t. Então, o Gasto Per capita relativo a 2008 é definido por:
* X(t)=(G(t)-G(2008))/pop08. 

* AS variáveis de controle serão: PIB pc relativo a 2008, transferencias FUNDEB per capita relativo a 2008, transferencias FPM pc relativa a 2008
* OBS: Outras variáveis de controle poderão ser adicionadas em exercícios de robustez

* Variável de Tratamento: 
* Transf_educ= Transferencia Relativa a cota parte do ICMS decorrente até 2008 do número de estudantes matriculados e após 2009 do IQE.

rename cotapartefpm fpm
rename TransferênciasdoFUNDEB fundeb

rename INDICEEDUCACAO iqe
rename INDICESAUDE iqs
rename INDICEMAMBIENTE iqm
destring SituaçãolimitesLRF, replace
rename SituaçãolimitesLRF situacao
rename Reeleição reeleicao



replace g_total= g_total/inf_dess
replace g_educ= g_educ/inf_dess
replace g_fund= g_fund/inf_dess
replace pib= pib/inf_dess
replace fpm= fpm/inf_dess
replace fundeb= fundeb/inf_dess
replace cp= cp/inf_dess
gen g_n_edu=g_total-g_educ

format %14.0g g_n_edu

save "D:\Pedro\Tese Pedro\Flypaper\flypaper-rafa\Base Inicial 2.dta", replace



keep if ano==2008
drop ano Municipio  iqe iqs iqm Soma_Indices BaseCalculoReceitaICMSR ICMS25 DeduçãoFUNDEB situacao reeleicao inf_dess id pib fpm fundeb

save "D:\Pedro\Tese Pedro\Flypaper\flypaper-rafa\base_08.dta", replace

rename pop pop08
rename g_total g_total08
rename g_n_edu g_n_edu08
rename g_educ g_educ08
rename g_fund g_fund08
rename cp cp08


*** Junção com base das escolas ***


save "D:\Pedro\Tese Pedro\Flypaper\flypaper-rafa\base_08.dta", replace


clear all
use "D:\Pedro\Tese Pedro\Flypaper\flypaper-rafa\Base Inicial 2.dta"
merge m:m cod_ibge using "D:\Pedro\Tese Pedro\Flypaper\flypaper-rafa\base_08.dta"

drop _merge

gen g_total_x=(g_total-g_total08)/pop08
gen g_nao_ed_x=(g_n_edu-g_n_edu08)/pop08
gen g_educ_x=(g_educ-g_educ08)/pop08
gen g_fund_x=(g_fund-g_fund08)/pop08
gen cp_x=(cp-cp08)/pop08


keep if ano==2009




egen rank_cp_x = xtile (cp_x), nq(3)

tab rank_cp_x, m


*7 missing -> São Luís do Curu, São Benedito, Cariré, Palmácia, Uruburetama, Ibaretama, Groaíras*
drop if  cod_ibge==2303105
drop if  cod_ibge==2310100
drop if  cod_ibge==2313807
drop if  cod_ibge==2312304
drop if  cod_ibge==2312601
drop if  cod_ibge==2304905
drop if  cod_ibge==2305266



drop pop-g_fund_x
drop ano Municipio

save "D:\Pedro\Tese Pedro\Flypaper\Pareamento\censo\cp_x.dta", replace


********************************************************************************
*******             UNINDO BASE ESCOLA + BASE COTA PARTE            ************
********************************************************************************


clear all


use "D:\Pedro\Tese Pedro\Flypaper\Pareamento\censo\spaece_inicial_1.dta"


merge m:m cod_ibge using "D:\Pedro\Tese Pedro\Flypaper\Pareamento\censo\cp_x.dta"

keep if _merge ==3


tab ano rank_cp_x
tab tratamento rank_cp_x, m
bysort ano: tab tratamento rank_cp_x, m


**********************************************************************************
* Criando ranks escola 


gen trat13=.
replace trat13 =1 if rank_07==1 & rank_cp_x==1
replace trat13 =0 if rank_07==1 & rank_cp_x==3

label variable trat13 " 1º quartil escola + 3º tercil cp"

gen trat12=.
replace trat12=1 if rank_07==1 & rank_cp_x==1
replace trat12=0 if rank_07==1 & rank_cp_x==2

label variable trat12 "1º quartil escola + 2º tercil cp"

gen trat11=.
replace trat11=1 if rank_07==1 & rank_cp_x==1
replace trat11=0 if rank_07==1 & rank_cp_x==2 
replace trat11=0 if rank_07==1 & rank_cp_x==3 
 
label variable trat11 "1º quartil escola + 2º e 3º tercil cp"


gen trat43=.
replace trat43=1 if rank_07==4 & rank_cp_x==1
replace trat43=0 if rank_07==4 & rank_cp_x==3

label variable trat43 "4º quartil escola + 3º tercil cp"


gen trat42=.
replace trat42=1 if rank_07==4 & rank_cp_x==1
replace trat42=0 if rank_07==4 & rank_cp_x==2

label variable trat42 "4º quartil escola + 2º tercil cp"


gen trat41=.
replace trat41=1 if rank_07==4 & rank_cp_x==1
replace trat41=0 if rank_07==4 & rank_cp_x==2 
replace trat41=0 if rank_07==4 & rank_cp_x==3 
 
label variable trat41 "4º quartil escola + 2º e 3º tercil cp"


********************************************************************

gen trat03=.
replace trat03=1 if rank_cp_x==3
replace trat03=0 if rank_cp_x==1

label variable trat03 "3º tercil cp em relação ao 1º tercil cp"


gen trat02 =.
replace trat02=1 if rank_cp_x==3
replace trat02=0 if rank_cp_x==2

label variable trat02 "3º tercil cp em relação ao 2º tercil cp"

save "D:\Pedro\Tese Pedro\Flypaper\Pareamento\censo\spaece_inicial_1.dta", replace

*** Drop nas escolas entrantes*


**** Estimações  ******

areg std_prof i.ano trat03, a(it) cl(cod_escola)
areg std_prof i.ano trat02, a(it) cl(cod_escola)


use "D:\Pedro\Tese Pedro\Flypaper\Pareamento\censo\spaece_inicial_1.dta"

