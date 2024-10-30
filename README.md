# conversoes-moedas
Relatório do Power BI contendo análise de transações em diferentes moedas


## dCalendario simples

```pq
let
    dataInicial = dataInicial, 
    dataFinal = dataFinal, 
    
    datas = List.Dates(
        dataInicial, 
        Duration.Days(dataFinal-dataInicial) + 1, 
        #duration(1, 0, 0, 0)
    ),

    calendario = #table(
        type table[
            Data = date,
            Ano = Int64.Type,
            MesAno = text,
            MesInicio = date
        ],
        List.Transform(
            datas,
            each {
                _,
                Date.Year(_),
                Date.ToText(_, [Format="MMM/yy", Culture="pt-BR"]),
                Date.StartOfMonth(_)
            }
        )
    )

in
    calendario
```

## Medidas criadas

```dax
Total R$ = SUM( fTransacoesReais[TotalReal] )
```


```dax
Total pela Moeda da Transacao = 
    SWITCH( TRUE(),
        
        SELECTEDVALUE( dMoedas[Moeda] ) = "BRL",  
            [Total R$], 
        
        ISINSCOPE( dMoedas[Moeda] ),
            SUMX(
                fCotacoes,
                DIVIDE( [Total R$], fCotacoes[Cotacao] )
            )
    )
```


```dax
Total pela Moeda Selecionada = 
    SWITCH( TRUE(),
        
        SELECTEDVALUE(dMoedas[Moeda] ) = "BRL", 
            CALCULATE([Total R$], REMOVEFILTERS(dMoedas)),
        
        ISINSCOPE(dMoedas[Moeda]),  
            SUMX(
                fCotacoes,
                DIVIDE( 
                    CALCULATE([Total R$], REMOVEFILTERS(dMoedas)),
                    fCotacoes[Cotacao] 
                )
            )
    )

```

```dax
Total R$ Recomposto = 

VAR __Real = 
    CALCULATE(
        SUM(fTransacaoesSemConversao[Total]),
        KEEPFILTERS(dMoedas[Moeda] = "BRL")
    )

VAR __Demais = 
    SUMX(
        fCotacoes,
        fCotacoes[Cotacao] 
        * 
        CALCULATE(
            SUM(fTransacaoesSemConversao[Total]),
            KEEPFILTERS(dMoedas[Moeda] <> "BRL")
        )
    ) 

RETURN 
    __Real + __Demais

```





