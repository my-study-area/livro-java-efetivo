# livro-java-efetivo
Anotações da leitura do Livro Java Efetivo: Terceira Edição

## 8. Métodos
As anotações desse capítulo valem tanto para métodos como para construtores.

**Item 49: Verifique a validade dos parâmetros**
- implementar as verificações dos parâmetros dos métodos o mais cedo possível, no início do corpo do método. O objeto somente deve ser alterado quando os parâmetros são válidos, evitando-se que o método deixe o objeto inconsiste para uso no futuro.

- as restrições devem ser documentadas. Use `@throws` para documentar as exceções. Exceções documentadas no nível de classe são validas para todos os métodos.

- realize verificações de nulidade usando `Objects.requiredNonNul(obj)`

- uma exceção da verificação explícita dos parâmetos é o processo de realização de calculos que ocorre de forma implícita. Como no caso da ordenação de uma lista de objetos, como o `Collections.sort(list)`, que caso, não sejam comparáveis lançam `ClassCastException`.
 