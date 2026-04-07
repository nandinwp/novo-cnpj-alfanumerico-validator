# novo-cnpj-alfanumerico-validator
Documentação: Validador de CNPJ Alfanumérico (IN RFB nº 2.229/2024)

# Guia de Implementacao Universal

Este documento descreve o novo padrao de CNPJ Alfanumerico implementado pela Receita Federal (Instrucao Normativa RFB n 2.229/2024), com entrada em vigor prevista para julho de 2026.

## Estrutura do Novo CNPJ
O CNPJ mantem 14 caracteres:
XX.XXX.XXX/XXXX-DD

- Posicoes 1 a 12 (Raiz e Ordem): Alfanumericas (A-Z e 0-9).
- Posicoes 13 e 14 (Digitos Verificadores): Sempre numericas (0-9).

## Algoritmo de Calculo (Modulo 11)

### 1. Conversao de Caracteres
- Digitos '0' a '9': Valor de 0 a 9.
- Letras 'A' a 'Z': Valor do codigo ASCII subtraindo 48 (Ex: 'A' = 17, 'B' = 18, ..., 'Z' = 42).

### 2. Pesos
- Primeiro Digito: 5, 4, 3, 2, 9, 8, 7, 6, 5, 4, 3, 2.
- Segundo Digito: 6, 5, 4, 3, 2, 9, 8, 7, 6, 5, 4, 3, 2.

### 3. Regra do Resto
Soma dos produtos dividida por 11.
- Se Resto for 0 ou 1: Digito Verificador = 0.
- Caso contrario: Digito Verificador = 11 - Resto.

---

## Implementacoes por Linguagem

### C
```c
#include <stdio.h>
#include <string.h>
#include <ctype.h>

int calcular_dv(const char* base, int* pesos, int len) {
    int soma = 0;
    for (int i = 0; i < len; i++) {
        int valor = (base[i] >= '0' && base[i] <= '9') ? (base[i] - '0') : (base[i] - 48);
        soma += valor * pesos[i];
    }
    int resto = soma % 11;
    return (resto < 2) ? 0 : 11 - resto;
}

int validar_cnpj(const char* cnpj) {
    if (strlen(cnpj) != 14) return 0;
    int p1[] = {5,4,3,2,9,8,7,6,5,4,3,2};
    int p2[] = {6,5,4,3,2,9,8,7,6,5,4,3,2};
    if (calcular_dv(cnpj, p1, 12) != (cnpj[12] - '0')) return 0;
    if (calcular_dv(cnpj, p2, 13) != (cnpj[13] - '0')) return 0;
    return 1;
}
```

### C++
```cpp
#include <string>
#include <vector>
#include <numeric>

bool isValidCnpj(std::string cnpj) {
    if (cnpj.length() != 14) return false;
    auto calc = [&](int n) {
        int soma = 0;
        std::vector<int> pesos = (n == 12) ? 
            std::vector<int>{5,4,3,2,9,8,7,6,5,4,3,2} : 
            std::vector<int>{6,5,4,3,2,9,8,7,6,5,4,3,2};
        for (int i = 0; i < n; i++) {
            int v = isdigit(cnpj[i]) ? cnpj[i] - '0' : cnpj[i] - 48;
            soma += v * pesos[i];
        }
        int r = soma % 11;
        return r < 2 ? 0 : 11 - r;
    };
    return calc(12) == (cnpj[12] - '0') && calc(13) == (cnpj[13] - '0');
}
```

### C#
```csharp
public static bool IsValidCnpj(string cnpj) {
    if (cnpj.Length != 14) return false;
    int Calcular(int n, int[] pesos) {
        int soma = 0;
        for (int i = 0; i < n; i++)
            soma += (char.IsDigit(cnpj[i]) ? cnpj[i] - '0' : cnpj[i] - 48) * pesos[i];
        int r = soma % 11;
        return r < 2 ? 0 : 11 - r;
    }
    return Calcular(12, new[] {5,4,3,2,9,8,7,6,5,4,3,2}) == (cnpj[12] - '0') &&
           Calcular(13, new[] {6,5,4,3,2,9,8,7,6,5,4,3,2}) == (cnpj[13] - '0');
}
```

### Java
```java
public static boolean isValidCnpj(String cnpj) {
    if (cnpj.length() != 14) return false;
    int[] p1 = {5,4,3,2,9,8,7,6,5,4,3,2};
    int[] p2 = {6,5,4,3,2,9,8,7,6,5,4,3,2};
    return calcular(cnpj, 12, p1) == Character.getNumericValue(cnpj.charAt(12)) &&
           calcular(cnpj, 13, p2) == Character.getNumericValue(cnpj.charAt(13));
}
private static int calcular(String s, int n, int[] p) {
    int sum = 0;
    for (int i = 0; i < n; i++) {
        char c = s.charAt(i);
        int v = Character.isDigit(c) ? c - '0' : c - 48;
        sum += v * p[i];
    }
    int r = sum % 11;
    return r < 2 ? 0 : 11 - r;
}
```

### Python
```python
def is_valid_cnpj(cnpj):
    if len(cnpj) != 14: return False
    def calc(s, pesos):
        soma = sum((ord(c) - 48 if not c.isdigit() else int(c)) * p for c, p in zip(s, pesos))
        resto = soma % 11
        return 0 if resto < 2 else 11 - resto
    p1 = [5,4,3,2,9,8,7,6,5,4,3,2]
    p2 = [6,5,4,3,2,9,8,7,6,5,4,3,2]
    return calc(cnpj[:12], p1) == int(cnpj[12]) and calc(cnpj[:13], p2) == int(cnpj[13])
```

### JavaScript / Node.js
```javascript
function isValidCnpj(cnpj) {
  if (cnpj.length !== 14) return false;
  const calc = (s, pesos) => {
    const soma = s.split('').reduce((acc, c, i) => {
      const v = /\d/.test(c) ? parseInt(c) : c.charCodeAt(0) - 48;
      return acc + v * pesos[i];
    }, 0);
    const r = soma % 11;
    return r < 2 ? 0 : 11 - r;
  };
  return calc(cnpj.substring(0, 12), [5,4,3,2,9,8,7,6,5,4,3,2]) === parseInt(cnpj[12]) &&
         calc(cnpj.substring(0, 13), [6,5,4,3,2,9,8,7,6,5,4,3,2]) === parseInt(cnpj[13]);
}
```

### PHP
```php
function isValidCnpj($cnpj) {
    if (strlen($cnpj) != 14) return false;
    $calc = function($s, $pesos) {
        $soma = 0;
        foreach (str_split($s) as $i => $c) {
            $v = is_numeric($c) ? (int)$c : ord($c) - 48;
            $soma += $v * $pesos[$i];
        }
        $r = $soma % 11;
        return $r < 2 ? 0 : 11 - $r;
    };
    return $calc(substr($cnpj, 0, 12), [5,4,3,2,9,8,7,6,5,4,3,2]) == $cnpj[12] &&
           $calc(substr($cnpj, 0, 13), [6,5,4,3,2,9,8,7,6,5,4,3,2]) == $cnpj[13];
}
```

### Go
```go
func IsValidCnpj(cnpj string) bool {
    if len(cnpj) != 14 { return false }
    calc := func(s string, pesos []int) int {
        soma := 0
        for i, c := range s {
            v := int(c - '0')
            if c >= 'A' && c <= 'Z' { v = int(c - 48) }
            soma += v * pesos[i]
        }
        r := soma % 11
        if r < 2 { return 0 }
        return 11 - r
    }
    return calc(cnpj[:12], []int{5,4,3,2,9,8,7,6,5,4,3,2}) == int(cnpj[12]-'0') &&
           calc(cnpj[:13], []int{6,5,4,3,2,9,8,7,6,5,4,3,2}) == int(cnpj[13]-'0')
}
```

### Kotlin
```kotlin
fun isValidCnpj(cnpj: String): Boolean {
    if (cnpj.length != 14) return false
    fun calc(s: String, p: IntArray): Int {
        val sum = s.withIndex().sumOf { (i, c) ->
            (if (c.isDigit()) c - '0' else c.code - 48) * p[i]
        }
        val r = sum % 11
        return if (r < 2) 0 else 11 - r
    }
    return calc(cnpj.take(12), intArrayOf(5,4,3,2,9,8,7,6,5,4,3,2)) == (cnpj[12] - '0') &&
           calc(cnpj.take(13), intArrayOf(6,5,4,3,2,9,8,7,6,5,4,3,2)) == (cnpj[13] - '0')
}
```

### Swift
```swift
func isValidCnpj(_ cnpj: String) -> Bool {
    guard cnpj.count == 14 else { return false }
    let chars = Array(cnpj)
    func calc(_ n: Int, _ weights: [Int]) -> Int {
        var sum = 0
        for i in 0..<n {
            let v = chars[i].isNumber ? Int(String(chars[i]))! : Int(chars[i].asciiValue!) - 48
            sum += v * weights[i]
        }
        let r = sum % 11
        return r < 2 ? 0 : 11 - r
    }
    return calc(12, [5,4,3,2,9,8,7,6,5,4,3,2]) == Int(String(chars[12])) &&
           calc(13, [6,5,4,3,2,9,8,7,6,5,4,3,2]) == Int(String(chars[13]))
}
```

### Rust
```rust
fn is_valid_cnpj(cnpj: &str) -> bool {
    if cnpj.len() != 14 { return false; }
    let b = cnpj.as_bytes();
    let calc = |n, p: &[i32]| {
        let sum: i32 = (0..n).map(|i| {
            let v = if b[i].is_ascii_digit() { (b[i] - b'0') as i32 } else { (b[i] - 48) as i32 };
            v * p[i]
        }).sum();
        let r = sum % 11;
        if r < 2 { 0 } else { 11 - r }
    };
    calc(12, &[5,4,3,2,9,8,7,6,5,4,3,2]) == (b[12] - b'0') as i32 &&
    calc(13, &[6,5,4,3,2,9,8,7,6,5,4,3,2]) == (b[13] - b'0') as i32
}
```

### Delphi / Object Pascal
```pascal
function IsValidCnpj(const cnpj: string): Boolean;
var
  sum, r, i: Integer;
  p1: array[0..11] of Integer = (5,4,3,2,9,8,7,6,5,4,3,2);
  p2: array[0..12] of Integer = (6,5,4,3,2,9,8,7,6,5,4,3,2);
  function Val(c: Char): Integer;
  begin
    if c in ['0'..'9'] then Result := Ord(c) - Ord('0') else Result := Ord(c) - 48;
  end;
begin
  if Length(cnpj) <> 14 then Exit(False);
  sum := 0;
  for i := 1 to 12 do sum := sum + Val(cnpj[i]) * p1[i-1];
  r := sum mod 11;
  if r < 2 then r := 0 else r := 11 - r;
  if r <> Val(cnpj[13]) then Exit(False);
  sum := 0;
  for i := 1 to 13 do sum := sum + Val(cnpj[i]) * p2[i-1];
  r := sum mod 11;
  if r < 2 then r := 0 else r := 11 - r;
  Result := r = Val(cnpj[14]);
end;
```

### Visual Basic .NET
```vb
Public Function IsValidCnpj(cnpj As String) As Boolean
    If cnpj.Length <> 14 Then Return False
    Dim calc = Function(n As Integer, p As Integer()) As Integer
        Dim sum As Integer = 0
        For i As Integer = 0 To n - 1
            Dim v = If(Char.IsDigit(cnpj(i)), Asc(cnpj(i)) - Asc("0"c), Asc(cnpj(i)) - 48)
            sum += v * p(i)
        Next
        Dim r = sum Mod 11
        Return If(r < 2, 0, 11 - r)
    End Function
    Return calc(12, {5,4,3,2,9,8,7,6,5,4,3,2}) = (Asc(cnpj(12)) - Asc("0"c)) And
           calc(13, {6,5,4,3,2,9,8,7,6,5,4,3,2}) = (Asc(cnpj(13)) - Asc("0"c))
End Function
```

### Perl
```perl
sub is_valid_cnpj {
    my ($cnpj) = @_;
    return 0 if length($cnpj) != 14;
    my @b = split //, $cnpj;
    my $calc = sub {
        my ($n, $p) = @_;
        my $sum = 0;
        for (my $i=0; $i<$n; $i++) {
            my $v = ($b[$i] =~ /\d/) ? $b[$i] : ord($b[$i]) - 48;
            $sum += $v * $p->[$i];
        }
        my $r = $sum % 11;
        return ($r < 2) ? 0 : 11 - $r;
    };
    return $calc->(12, [5,4,3,2,9,8,7,6,5,4,3,2]) == $b[12] &&
           $calc->(13, [6,5,4,3,2,9,8,7,6,5,4,3,2]) == $b[13];
}
```

### R
```r
isValidCnpj <- function(cnpj) {
  if (nchar(cnpj) != 14) return(FALSE)
  chars <- strsplit(cnpj, "")[[1]]
  calc <- function(n, p) {
    vals <- sapply(chars[1:n], function(x) {
      if (grepl("[0-9]", x)) as.numeric(x) else utf8ToInt(x) - 48
    })
    soma <- sum(vals * p)
    r <- soma %% 11
    if (r < 2) 0 else 11 - r
  }
  return(calc(12, c(5,4,3,2,9,8,7,6,5,4,3,2)) == as.numeric(chars[13]) &&
         calc(13, c(6,5,4,3,2,9,8,7,6,5,4,3,2)) == as.numeric(chars[14]))
}
```

### Matlab
```matlab
function valid = isValidCnpj(cnpj)
    if length(cnpj) ~= 14, valid = false; return; end
    calc = @(n, p) mod11(cnpj(1:n), p);
    valid = calc(12, [5,4,3,2,9,8,7,6,5,4,3,2]) == (cnpj(13)-'0') && ...
            calc(13, [6,5,4,3,2,9,8,7,6,5,4,3,2]) == (cnpj(14)-'0');
end
function dv = mod11(s, p)
    vals = arrayfun(@(c) charVal(c), s);
    soma = sum(vals .* p);
    r = mod(soma, 11);
    if r < 2, dv = 0; else, dv = 11 - r; end
end
function v = charVal(c)
    if isstrprop(c, 'digit'), v = c - '0'; else, v = c - 48; end
end
```

### Fortran
```fortran
function is_valid_cnpj(cnpj)
    character(len=14), intent(in) :: cnpj
    logical :: is_valid_cnpj
    integer :: p1(12) = [5,4,3,2,9,8,7,6,5,4,3,2], p2(13) = [6,5,4,3,2,9,8,7,6,5,4,3,2]
    is_valid_cnpj = (calc(12, p1) == iachar(cnpj(13:13))-48) .and. (calc(13, p2) == iachar(cnpj(14:14))-48)
contains
    integer function calc(n, p)
        integer, intent(in) :: n, p(n)
        integer :: i, v, soma
        soma = 0
        do i = 1, n
            if (cnpj(i:i) >= '0' .and. cnpj(i:i) <= '9') then
                v = iachar(cnpj(i:i)) - 48
            else
                v = iachar(cnpj(i:i)) - 48
            end if
            soma = soma + v * p(i)
        end do
        calc = mod(soma, 11)
        if (calc < 2) then; calc = 0; else; calc = 11 - calc; endif
    end function
end function
```

### Groovy / Grails
```groovy
static boolean isValidCnpj(String cnpj) {
    if (cnpj?.length() != 14) return false
    def calc = { s, p ->
        int sum = 0
        s.eachWithIndex { c, i ->
            int v = c.isDigit() ? c.toInteger() : (int)c[0] - 48
            sum += v * p[i]
        }
        int r = sum % 11
        r < 2 ? 0 : 11 - r
    }
    return calc(cnpj[0..11], [5,4,3,2,9,8,7,6,5,4,3,2]) == cnpj[12].toInteger() &&
           calc(cnpj[0..12], [6,5,4,3,2,9,8,7,6,5,4,3,2]) == cnpj[13].toInteger()
}
```

### Ada
```ada
function Is_Valid_Cnpj(Cnpj : String) return Boolean is
    type Int_Array is array (Positive range <>) of Integer;
    P1 : constant Int_Array := (5,4,3,2,9,8,7,6,5,4,3,2);
    P2 : constant Int_Array := (6,5,4,3,2,9,8,7,6,5,4,3,2);
    function Calc(N : Positive; P : Int_Array) return Integer is
        Sum : Integer := 0;
        Val : Integer;
    begin
        for I in 1 .. N loop
            if Cnpj(I) in '0' .. '9' then
                Val := Character'Pos(Cnpj(I)) - Character'Pos('0');
            else
                Val := Character'Pos(Cnpj(I)) - 48;
            end if;
            Sum := Sum + Val * P(I);
        end loop;
        declare
            R : Integer := Sum mod 11;
        begin
            return (if R < 2 then 0 else 11 - R);
        end;
    end Calc;
begin
    return Cnpj'Length = 14 and then
           Calc(12, P1) = Character'Pos(Cnpj(13)) - Character'Pos('0') and then
           Calc(13, P2) = Character'Pos(Cnpj(14)) - Character'Pos('0');
end Is_Valid_Cnpj;
```
---

## Referencias
- Instrucao Normativa RFB n 2.229/2024.
- Codigo Fonte TV (Gabriel Froes e Vanessa Weber).

