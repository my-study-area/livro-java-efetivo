# livro-java-efetivo
Anotações da leitura do Livro Java Efetivo: Terceira Edição

## 8. Métodos
As anotações desse capítulo valem tanto para métodos como para construtores.

**Item 49: Verifique a validade dos parâmetros**
- implementar as verificações dos parâmetros dos métodos o mais cedo possível, no início do corpo do método. O objeto somente deve ser alterado quando os parâmetros são válidos, evitando-se que o método deixe o objeto inconsiste para uso no futuro.

- as restrições devem ser documentadas. Use `@throws` para documentar as exceções. Exceções documentadas no nível de classe são validas para todos os métodos.
```java
//BOM
/**
  * Retorna uma BigInteger cujo valor é (this mod m).  Esse método
  * difere do método restante na medida em que ele sempre retorna uma
  * BigInteger não negativa.
  *
  * o @param m modulo, deve ser positivo
  * @return seu mod. m
  * lança uma @throws ArithmeticException se m for menor ou igual a 0
  */
public BigInteger mod(BigInteger m) {
    if (m.signum <= 0)
        throw new ArithmeticException("BigInteger: modulus not positive");

    // restante do código
}
```

- realize verificações de nulidade usando `Objects.requiredNonNul(obj)`
```java
Objects.requireNonNull(obj, "Descrição da exception");
```

- uma exceção da verificação explícita dos parâmetos é o processo de realização de calculos que ocorre de forma implícita. Como no caso da ordenação de uma lista de objetos, como o `Collections.sort(list)`, que caso, não sejam comparáveis lançam `ClassCastException`.
 
**Item 50: Faça cópias defensivas quando necessário**
- programe defensivamente, partindo do princípio que os clientes de sua classe farão o melhor para destruir as invariantes dela.

- é fundamental fazer uma cópia defensiva de cada parâmetro `mutável` para seu contrutor

```java
//RUIM
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentExcpetion();
        this.start = start;
        this.end = end;
    }

    public Date start() {
        return start;
    }

    public Date end() {
        return end;
    }
}

// Exemplo de ataque:
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
System.out.println(p.end()); //Thu Feb 11 21:39:29 BRT 2021
end.setYear(78);
System.out.println(p.end()); //Sat Feb 11 21:39:29 BRT 1978
```
```java
//BOM
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        //cópia defensiva
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());

        if (start.compareTo(end) > 0)
            throw new IllegalArgumentExcpetion();
    }

    public Date start() {
        return start;
    }

    public Date end() {
        return end;
    }
}

// Tentativa de ataque sem sucesso
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
System.out.println(p.end()); //Thu Feb 11 21:48:38 BRT 2021
end.setYear(78);
System.out.println(p.end()); //Thu Feb 11 21:48:38 BRT 2021
```
- não use o método clone para fazer uma cópia defensiva de um parâmetro cujo tipo possa ser subclasseado por terceiros não confiáveis

```java
class MaliciousDate extends Date {
    private final List<MaliciousDate> dates;
    public MaliciousDate(List<MaliciousDate> dates) {
        this.dates = dates;
    }
    @Override public MaliciousDate clone() {
        MaliciousDate other = (MaliciousDate) super.clone(); //Ou new MalicousDate
        synchronized (dates) {
            dates.add(other);
        }
        return other; //Ou returna this;
    }
}

//RUIM
public Period(Date start, Date end) {
    //Falha na cópia defensiva.
    start = (Date)start.clone();
    end   = (Date)end  .clone();

    if (start.compareTo(end) > 0)
        throw new IllegalArgumentExcpetion();
    this.start = start;
    this.end = end;
}

//Exemplo de ataque
List<MaliciousDate> dates = new ArrayList<>();
Date start = new MaliciousDate(dates);
Date end = new MaliciousDate(dates);
Period p = new Period(start, end);
System.out.println(p.end()); //Fri Feb 12 14:23:59 BRT 2021
dates.get(1).setYear(78); //modifica o internals do objeto p
System.out.println(p.end()); //Sun Feb 12 14:23:59 BRT 1978
```
Obs: _Exemplo de código encontrado no [stackoverflow](https://stackoverflow.com/a/21901867/6415045)_
