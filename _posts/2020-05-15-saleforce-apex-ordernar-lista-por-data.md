---
layout: post
title:  "Saleforce - Apex - Ordernar Lista por data"
date:   2020-05-15 00:57:00 +0300
categories: Javascript NodeJS
---

E aí pessoal, tudo bem?

Hoje vou falar sobre como ordenar uma lista no Apex por data.

Estou baseando esse post em algo que aconteceu no meu trabalho do dia a dia onde surgiu a necessidade de descobrir qual o maior intervalo de meses entre séries de pagamento.

Basicamente tínhamos um cliente realizando uma compra de um imóvel sendo que ele realizaria o pagamento de algumas formas diferentes:

| # |Série|Início|Quantidade Parcelas|Periodicidade (em meses)|
|---|---|---|---|---|
|1|Entrada |07/2020|1|1|
|2|Parcelas Mensais|01/2021|120|1|
|3|Parcelas Anuais|12/2021|10|12|

Então surgiu a necessidade de apurar qual o intervalo de meses onde o cliente não realizaria nenhum pagamento. Por exemplo, se a entrada é no mês 07/2020 e as parcelas mensais iniciam apenas em 01/2021, temos 5 meses onde não é realizado nenhum pagamento.

Então eu fiz uma linha do tempo onde eu incluí todos os meses que existem parcelas a serem pagas, porém como a geração das parcelas é por série de pagamento, os meses não ficavam na ordem sequencial, e então surgiu a necessidade de reordernar a lista inteira de forma sequencial.

E é aí que entra o método ```Comparable``` - [Ver na documentação do salesforce][Comparable]

Basicamente vamos sobrescrever o método ```sort``` da lista, por um novo método onde iremos definir qual é a ordem que ele deve obedecer para ordernar os registros.

No código abaixo criamos uma classe onde recebemos a lista com as datas para reordenar no método ```findInterval```, que por sua vez irá iniciar uma lista da classe ```PaymentsTimeline``` que implementa o ```Comparable```.

Fiz também a inclusão de duas linhas de depuração, antes e depois da ordenação.

```java
public class AnyService {

  public static void findInterval(List<Date> paymentDates){

    List<PaymentsTimeline> payments = new List<PaymentsTimeline>();
    
    for(Date d : paymentDates){
      PaymentsTimeline pt = new PaymentsTimeline();
      pt.paymentDate = d;
      payments.add(pt);
    }

    System.debug(payments);

    payments.sort();

    System.debug(payments);

  }

  public class PaymentsTimeline implements Comparable {
    Date paymentDate;

    public Integer compareTo(Object instance) {
        PaymentsTimeline that = (PaymentsTimeline) instance;
        if (this.paymentDate < that.paymentDate) return -1;
        if (this.paymentDate > that.paymentDate) return 1;
        return 0; 
    }
  }
}
```

### Testando

Executei o código abaixo no modo de execução anônima de código do developer console e marquei a opção "Open Log" para poder ver as linhas de depuração que configuramos na classe.

```java
List<Date> dates = new List<Date>();

dates.add(Date.newInstance(2022,07,15));
dates.add(Date.newInstance(2021,02,15));
dates.add(Date.newInstance(2020,05,15));
dates.add(Date.newInstance(2016,12,15));
dates.add(Date.newInstance(2024,08,15));
dates.add(Date.newInstance(2022,06,15));

AnyService.findInterval(dates);
```

Percebam que as datas estão de totalmente aleatórias e sem nenhuma lineariedade em relação ao tempo.

Agora vamos aos resultados do log.

##### Linha 1 - Antes da ordenação - Datas na ordem que foram enviadas
```
DEBUG
PaymentsTimeline:[paymentDate=2022-07-15 00:00:00],
PaymentsTimeline:[paymentDate=2021-02-15 00:00:00],
PaymentsTimeline:[paymentDate=2020-05-15 00:00:00],
PaymentsTimeline:[paymentDate=2016-12-15 00:00:00],
PaymentsTimeline:[paymentDate=2024-08-15 00:00:00],
PaymentsTimeline:[paymentDate=2022-06-15 00:00:00]
```

##### Linha 2 - Após a ordenação - Datas ordenadas corretamente
```
DEBUG
PaymentsTimeline:[paymentDate=2016-12-15 00:00:00],
PaymentsTimeline:[paymentDate=2020-05-15 00:00:00],
PaymentsTimeline:[paymentDate=2021-02-15 00:00:00],
PaymentsTimeline:[paymentDate=2022-06-15 00:00:00],
PaymentsTimeline:[paymentDate=2022-07-15 00:00:00],
PaymentsTimeline:[paymentDate=2024-08-15 00:00:00]
```

E é isso, se você ainda não conhecia, agora conhece uma forma interessante de ordenar listas de diferentes maneiras utilizando o ```Comparable```.

Um grande abraço e até a próxima.

Se você tem alguma pergunta pode me procurar nas redes sociais ou no email lucas@valhos.com.

[Comparable]: https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_comparable.htm
