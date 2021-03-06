// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © HeWhoMustNotBeNamed

//@version=4
study("Quality Screen", overlay=true)

Position = input(title="Table Position", defval=position.bottom_right,
                         options=[
                         position.bottom_center, position.bottom_left, position.bottom_right,
                         position.middle_center, position.middle_left, position.middle_right,
                         position.top_center, position.top_left, position.top_right
                         ])
textSize = input(title="Text Size", defval=size.small, options=[size.huge, size.large, size.normal, size.small, size.tiny, size.auto])
period = "FY"

f_getFinancials(financial_id, period) => financial(syminfo.tickerid, financial_id, period)
f_getColorByScore(score) => (score == 2 ? color.green : score == 1? color.lime : score == 0? color.silver : score == -1? color.orange : score == -2? color.red : color.purple)
f_getOptimalFinancialBasic(financial_id) =>
    f_getFinancials(financial_id, syminfo.currency == "USD"? "FQ" : "FY")
    
if barstate.islast
    deq = f_getOptimalFinancialBasic("DEBT_TO_EQUITY")
    dta = f_getOptimalFinancialBasic("DEBT_TO_ASSET")
    ldta = f_getOptimalFinancialBasic("LONG_TERM_DEBT_TO_ASSETS")
    altzscore = f_getOptimalFinancialBasic("ALTMAN_Z_SCORE")
    springate = f_getFinancials("SPRINGATE_SCORE", "FY")
    
    currentRatio = f_getOptimalFinancialBasic("CURRENT_RATIO")
    quickRatio = f_getOptimalFinancialBasic("QUICK_RATIO")
    sloanRatio = f_getOptimalFinancialBasic("SLOAN_RATIO")
    interestCover = f_getOptimalFinancialBasic("INTERST_COVER")
    
    roa = f_getOptimalFinancialBasic("RETURN_ON_ASSETS")
    roe = f_getOptimalFinancialBasic("RETURN_ON_EQUITY")
    roic = f_getOptimalFinancialBasic("RETURN_ON_INVESTED_CAPITAL")
    netMargin = f_getFinancials("NET_MARGIN", "FY")
    fcfMargin = f_getFinancials("FREE_CASH_FLOW_MARGIN", "FY")
    
    pfScore = f_getOptimalFinancialBasic("PIOTROSKI_F_SCORE")
    sgr = f_getOptimalFinancialBasic("SUSTAINABLE_GROWTH_RATE")
    
    orientation = Position == position.bottom_center or Position == position.middle_center or Position == position.top_center? 2:
                     Position == position.bottom_left or Position == position.middle_left or Position == position.top_left? 1 : 3
    
    var financialTable = table.new(position = Position, columns = 10, rows = 18, border_width = 1)
    columnSolvency = orientation == 1? 2 : orientation == 3? 0 : 0
    columnProfitability = orientation == 1? 2 : orientation == 3? 0 : 2
    columnLiquidity = orientation == 1? 2 : orientation == 3? 0 : 4
    columnGrowth = orientation == 1? 2 : orientation == 3? 0 : 6

    rowSolvency = orientation == 2? 0 : 0
    rowProfitability = orientation == 2? 0 : 5
    rowLiquidity = orientation == 2? 0 : 10
    rowGrowth = orientation == 2? 0 : 14

    columnValSolvency = orientation == 1? 0 : orientation == 3? 1 : 0
    columnValProfitability = orientation == 1? 0 : orientation == 3? 1 : 2
    columnValLiquidity = orientation == 1? 0 : orientation == 3? 1 : 4
    columnValGrowth = orientation == 1? 0 : orientation == 3? 1 : 6
    
    rowValSolvency = orientation == 2? 1 : 0
    rowValProfitability = orientation == 2? 1 : 5
    rowValLiquidity = orientation == 2? 1 : 10
    rowValGrowth = orientation == 2? 1 : 14
    
    table.cell(table_id = financialTable, column = columnSolvency, row = rowSolvency , text = "Solvency", bgcolor=color.black, text_color=color.white, text_size=textSize)
    table.cell(table_id = financialTable, column = columnProfitability, row = rowProfitability, text = "Profitability", bgcolor=color.black, text_color=color.white, text_size=textSize)
    table.cell(table_id = financialTable, column = columnLiquidity, row = rowLiquidity, text = "Liquidity", bgcolor=color.black, text_color=color.white, text_size=textSize)
    table.cell(table_id = financialTable, column = columnGrowth, row = rowGrowth, text = "Value/Growth", bgcolor=color.black, text_color=color.white, text_size=textSize)

    deq_score = na(deq)? 0 : deq > 0.0 and deq <= 1.0 ? 2 : deq > 1.0 and deq <= 2.0 ? 1 : deq > 2 ? -1 : -2
    table.cell(table_id = financialTable, column = columnValSolvency, row = rowValSolvency, text = "DEBT_TO_EQUITY", bgcolor=f_getColorByScore(deq_score), text_size=textSize)
    table.cell(table_id = financialTable, column = columnValSolvency+1, row = rowValSolvency, text = tostring(deq), bgcolor=f_getColorByScore(deq_score), text_size=textSize)

    dta_score = na(dta)? 0 : dta > 0.0 and dta <= 0.2 ? 2 : dta > 0.2 and dta <= 0.4 ? 1 : dta > 0.4 and dta <= 0.6 ? 0 : dta > 0.6 ? 1 : -2
    table.cell(table_id = financialTable, column = columnValSolvency, row = rowValSolvency+1, text = "DEBT_TO_ASSET", bgcolor=f_getColorByScore(dta_score), text_size=textSize)
    table.cell(table_id = financialTable, column = columnValSolvency+1, row = rowValSolvency+1, text = tostring(dta), bgcolor=f_getColorByScore(dta_score), text_size=textSize)

    ldta_score = na(ldta)? 0 : ldta > 0.0 and ldta <= 0.2 ? 2 : ldta > 0.2 and ldta <= 0.4 ? 1 : ldta > 0.4 and ldta <= 0.6 ? 0 : ldta > 0.6 ? 1 : -2
    table.cell(table_id = financialTable, column = columnValSolvency, row = rowValSolvency+2, text = "LONG_TERM_DEBT_TO_ASSETS", bgcolor=f_getColorByScore(ldta_score), text_size=textSize)
    table.cell(table_id = financialTable, column = columnValSolvency+1, row = rowValSolvency+2, text = tostring(ldta), bgcolor=f_getColorByScore(ldta_score), text_size=textSize)

    altz_score = na(altzscore)? 0 : altzscore > 5.0 ? 2 : altzscore > 3.0 ? 1 : altzscore > 1.8? 0 : altzscore > 1 ? -1 : -2
    table.cell(table_id = financialTable, column = columnValSolvency, row = rowValSolvency+3, text = "ALTMAN_Z_SCORE", bgcolor=f_getColorByScore(altz_score), text_size=textSize)
    table.cell(table_id = financialTable, column = columnValSolvency+1, row = rowValSolvency+3, text = tostring(altzscore), bgcolor=f_getColorByScore(altz_score), text_size=textSize)

    springate_score = na(springate)? 0 : springate > 0.862 ? 1 : -1
    table.cell(table_id = financialTable, column = columnValSolvency, row = rowValSolvency+4, text = "SPRINGATE_SCORE", bgcolor=f_getColorByScore(springate_score), text_size=textSize)
    table.cell(table_id = financialTable, column = columnValSolvency+1, row = rowValSolvency+4, text = tostring(springate), bgcolor=f_getColorByScore(springate_score), text_size=textSize)
    
    roe_score = na(roe)? 0 : roe >= 40? 2 : roe >=10 ? 1 : roe >=0 ? 0 : roe >= -20? -1 : -2
    table.cell(table_id = financialTable, column = columnValProfitability, row = rowValProfitability, text = "RETURN_ON_EQUITY", bgcolor=f_getColorByScore(roe_score), text_size=textSize)
    table.cell(table_id = financialTable, column = columnValProfitability+1, row = rowValProfitability, text = tostring(roe), bgcolor=f_getColorByScore(roe_score), text_size=textSize)
    
    roa_score = na(roa)? 0 : roa >= 20? 2 : roa >=5 ? 1 : roa >=0 ? 0 : roa >= -10? -1 : -2
    table.cell(table_id = financialTable, column = columnValProfitability, row = rowValProfitability+1, text = "RETURN_ON_ASSETS", bgcolor=f_getColorByScore(roa_score), text_size=textSize)
    table.cell(table_id = financialTable, column = columnValProfitability+1, row = rowValProfitability+1, text = tostring(roa), bgcolor=f_getColorByScore(roa_score), text_size=textSize)
    
    roic_score = na(roic)? 0 : roic >= 10? 2 : roic >=2 ? 1 : roic >=0 ? 0 : roic >= -5? -1 : -2
    table.cell(table_id = financialTable, column = columnValProfitability, row = rowValProfitability+2, text = "RETURN_ON_INVESTED_CAPITAL", bgcolor=f_getColorByScore(roic_score), text_size=textSize)
    table.cell(table_id = financialTable, column = columnValProfitability+1, row = rowValProfitability+2, text = tostring(roic), bgcolor=f_getColorByScore(roic_score), text_size=textSize)
    
    netMargin_score = na(netMargin)? 0 : netMargin >= 20? 2 : netMargin >=10 ? 1 : netMargin >=0 ? 0 : netMargin >= -5? -1 : -2
    table.cell(table_id = financialTable, column = columnValProfitability, row = rowValProfitability+3, text = "NET_MARGIN", bgcolor=f_getColorByScore(netMargin_score), text_size=textSize)
    table.cell(table_id = financialTable, column = columnValProfitability+1, row = rowValProfitability+3, text = tostring(netMargin), bgcolor=f_getColorByScore(netMargin_score), text_size=textSize)
    
    fcfMargin_score = na(fcfMargin)? 0 : fcfMargin >= 15? 2 : fcfMargin >=10 ? 1 : fcfMargin >=0 ? 0 : fcfMargin >= -10? -1 : -2
    table.cell(table_id = financialTable, column = columnValProfitability, row = rowValProfitability+4, text = "FREE_CASH_FLOW_MARGIN", bgcolor=f_getColorByScore(fcfMargin_score), text_size=textSize)
    table.cell(table_id = financialTable, column = columnValProfitability+1, row = rowValProfitability+4, text = tostring(fcfMargin), bgcolor=f_getColorByScore(fcfMargin_score), text_size=textSize)
    
    currentRatio_score = na(currentRatio)? 0 : currentRatio >= 1.2? (currentRatio <= 2.0 ? 2 : 1) : currentRatio >=1 ? 0 : currentRatio >=0.5 ? -1 : -2
    table.cell(table_id = financialTable, column = columnValLiquidity, row = rowValLiquidity, text = "CURRENT_RATIO", bgcolor=f_getColorByScore(currentRatio_score), text_size=textSize)
    table.cell(table_id = financialTable, column = columnValLiquidity+1, row = rowValLiquidity, text = tostring(currentRatio), bgcolor=f_getColorByScore(currentRatio_score), text_size=textSize)
    
    quickRatio_score = na(quickRatio)? 0 : quickRatio >= 1.0? (quickRatio <= 2.0 ? 2 : 1) : quickRatio >=0.9 ? 0 : currentRatio >=0.4 ? -1 : -2
    table.cell(table_id = financialTable, column = columnValLiquidity, row = rowValLiquidity+1, text = "QUICK_RATIO", bgcolor=f_getColorByScore(quickRatio_score), text_size=textSize)
    table.cell(table_id = financialTable, column = columnValLiquidity+1, row = rowValLiquidity+1, text = tostring(quickRatio), bgcolor=f_getColorByScore(quickRatio_score), text_size=textSize)
    
    sloanRatio_score = na(sloanRatio)? 0 : abs(sloanRatio) < 10.0? 1 : abs(sloanRatio) < 25.0? 0 : -1
    table.cell(table_id = financialTable, column = columnValLiquidity, row = rowValLiquidity+2, text = "SLOAN_RATIO", bgcolor=f_getColorByScore(sloanRatio_score), text_size=textSize)
    table.cell(table_id = financialTable, column = columnValLiquidity+1, row = rowValLiquidity+2, text = tostring(sloanRatio), bgcolor=f_getColorByScore(sloanRatio_score), text_size=textSize)
    
    interestCover_score = na(interestCover)? 0 : interestCover >3? 1 : interestCover >2? 0: -1
    table.cell(table_id = financialTable, column = columnValLiquidity, row = rowValLiquidity+3, text = "INTERST_COVER", bgcolor=f_getColorByScore(interestCover_score), text_size=textSize)
    table.cell(table_id = financialTable, column = columnValLiquidity+1, row = rowValLiquidity+3, text = tostring(interestCover), bgcolor=f_getColorByScore(interestCover_score), text_size=textSize)
    
    pf_score = na(pfScore)? 0 : pfScore >=8? 2 : pfScore >= 5 ? 1: pfScore >2 ? 0 : pfScore > 1? -1 : -2
    table.cell(table_id = financialTable, column = columnValGrowth, row = rowValGrowth, text = "PIOTROSKI_F_SCORE", bgcolor=f_getColorByScore(pf_score), text_size=textSize)
    table.cell(table_id = financialTable, column = columnValGrowth+1, row = rowValGrowth, text = tostring(pfScore), bgcolor=f_getColorByScore(pf_score), text_size=textSize)
    
    sgr_score = na(sgr)? 0 : sgr > 10? 2 : sgr > 5? 1 : sgr > 0? 0 : sgr > -5 ? -1 : -2
    table.cell(table_id = financialTable, column = columnValGrowth, row = rowValGrowth+1, text = "SUSTAINABLE_GROWTH_RATE", bgcolor=f_getColorByScore(sgr_score), text_size=textSize)
    table.cell(table_id = financialTable, column = columnValGrowth+1, row = rowValGrowth+1, text = tostring(sgr), bgcolor=f_getColorByScore(sgr_score), text_size=textSize)
