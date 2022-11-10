# Linux vs PowerShell  

Esto en Linux ` cut -d '@' -f1 ` es lo mismo que ` $linea.split("@")[0] ` en PowerShell. Sirve para trocear una cadena en diferentes partes seccionadas según el símbolo en este caso '@' puede ser cualquier otro también.

` "cadena de texto que en realidad es un comando" | iex ` --> usado para transformar texto en comandos.  

` (Get-WmiObject -Class Win32_Thread) ` es casi lo mismo que ` (Get-Process).Threads `  

` Write-Host ` para escribir por consola.  

De manera simbólica ` %{ "comandos" } ` equivale a un ` foreach(elem in lista) { "comandos" } ` de forma compacta  

El símbolo ` $_ ` para la manera simbólica equivale a  ` elem ` en un _foreach_ que es el elemento u objeto de la ciclo actual.  

```Bash
ps - report a snapshot of the current processes. 
 -f - Do full-format listing. When used with -L, the NLWP (number of threads) and LWP (thread ID) columns will be added.
 -L - Show threads, possibly with LWP and NLWP columns.
 -fL - Leer nombre de proceso de un fichero y mostrar hilos de una sola tacada.
 -C cmdlist - Select by command name.  This selects the processes whose executable name is given in cmdlist.  
 --pid pidlist - Select by process ID.
```

` "resultado de comandos anteriores" | Out-File "nombre del fichero destino" ` para guardar el resultado en un fichero.  
` Out-File fichero ` equivale a ` > fichero ` y si quieres seguir escribiendo sin sobreescribir ` >> fichero ` sigue por donde iba.  



--- 
--- 

# ANALIZAR HILOS CON POWERSHELL (POWERSHELL, HILOS)

## listar hilos mediante Get-Process
```PowerShell
get-process | select name,threads
```

## listar hilos mediante la llamada WMI Win32_Thread
```PowerShell
Get-WmiObject -Class Win32_Thread | Select-Object -First 20
```

## Prioridad de los hilos de los procesos (opción 1)
```PowerShell
(Get-WmiObject -Class Win32_Thread) | Select-Object ProcessHandle,Priority,PriorityBase | Sort-Object ProcessHandle
```

## Prioridad de los hilos de los procesos (opción 2)
```PowerShell
(Get-Process).Threads | Select-Object Id,CurrentPriority,BasePriority | Sort-Object Id
```

## Conocer el estado de los hilos de los procesos (opción 1)
```PowerShell
(Get-WmiObject -Class Win32_Thread) | Select-Object ProcessHandle,ThreadState | Sort-Object ProcessHandle
```
 
## Conocer el estado de los hilos de los procesos (opción 2)
```PowerShell
(Get-Process).Threads | Select-Object Id,ThreadState | Sort-Object Id
```

## Mostrar los hilos que ejecuta cada proceso
```PowerShell
Get-Process | %{
Write-Host $_.Name($_.Threads).Id
}
```

## Mostrar el número de hilos que ejecuta cada proceso
```PowerShell
Get-Process | %{
Write-Host $_.Name($_.Threads).Count
}
```

## Mostrar el proceso que creó el hilo
```PowerShell
(Get-WmiObject -Class Win32_Thread).ProcessHandle | %{
Get-Process -Id $_
}
```

------------
------------

# Resolver ejercicio propuesto: listar los procesos cuyo identificador de hilo se encuentre en una lista (el listado de procesos aparece escrito en un fichero y se tiene que ejecutar con IEX) (el fichero fichero.txt tiene Get-Process como contenido y el fichero hilo.txt tiene un número como contenido)
```PowerShell
(Get-Content .\fichero.txt) + " -id (Get-WmiObject win32_thread | where handle -eq (Get-Content .\hilo.txt)).ProcessHandle" | iex
```

-----------------
-----------------

# Examen mes de noviembre (obtener información sobre hilos en Linux y PowerShell)

## Linux

### Leer nombre de proceso de un fichero y mostrar hilos
```Bash
echo "ps -fL -C @sh" > fichero
echo "ps -fL -C @sh" >> fichero
while read linea
do
echo $linea
`echo $linea | cut -d '@' -f1 ; echo $linea | cut -d '@' -f2` >> resultado
done < fichero
cat resultado
```

### Mostrar el nombre de los procesos leyendo de un fichero el identificador de proceso
```Bash
echo "323@232@8888" > fichero
while read linea
do
echo $linea
ps -f -p `echo $linea | cut -d '@' -f1` >> resultado
done < fichero
cat resultado
```

### Sacar los hilos de los procesos leyendo el nombre del proceso de un fichero
```Bash
echo "nano" > fichero
echo "sh" >> fichero
while read linea
do
echo $linea
ps -fL -C `echo $linea | cut -d '@' -f1` >> resultado
done < fichero
cat resultado
```

## PowerShell

### Leer nombre de proceso de un fichero y mostrar hilos
```PowerShell
echo "ps @notepad" > fichero
echo "ps @notepad" >> fichero
Get-Content .\fichero
foreach($linea in Get-Content .\fichero)
{
    $ejecutar = $linea.split("@")[0] + $linea.split("@")[1]
    $ejecutar + " | select threads" | iex
}
```

### Mostrar el nombre de los procesos leyendo de un fichero el identificador de proceso
```PowerShell
echo "1288" > fichero
echo "1200" >> fichero
Get-Content .\fichero
foreach($linea in Get-Content .\fichero)
{
    Get-Process -Id $linea | select name
}
```

### Sacar los hilos de los procesos leyendo el nombre del proceso de un fichero
```PowerShell
echo "notepad" > fichero
echo "chrome" >> fichero
Get-Content .\fichero
foreach($linea in Get-Content .\fichero)
{
    Get-Process -name $linea | select name,threads
}
```

### Analizar hilos (solución)
https://www.jesusninoc.com/02/06/analizar-hilos-con-powershell/

### Otrá más
```PowerShell
# Listar identificadores de hilos
(Get-Process -name notepad | select threads).threads.id
# Listar información de un hilo en concreto
Get-WmiObject -Class win32_thread -Filter "Handle = 6248" | Out-File
# Sirve para analizar la información de hilos en función del proceso que los crea (Notepad)
(Get-Process -name notepad).id
$var=(Get-Process -name notepad).id
Get-WmiObject -Class win32_thread -Filter "ProcessHandle = $var"
# (Leyenda) Sirve para analizar información de hilos en función de los identificadores de cada hilo del proceso que los crea (Notepad)
$var=(Get-Process -name notepad | select threads).threads.id
$var | %{$_}
Get-WmiObject -Class win32_thread -Filter "ProcessHandle = $var"
$var | %{Get-WmiObject -Class win32_thread -Filter "Handle = $_"}
```

### Otra posible solución
```PowerShell
#Mostrar el proceso que creó el hilo
Get-WmiObject -Class Win32_Thread | %{
    if((Get-Process -Id $_.ProcessHandle).name -eq "notepad")
    {
        write-host $_.ProcessHandle,$_.handle
        Get-Process -Id $_.ProcessHandle | select name,id
    }
}
```

### Solución para entender la solución de Linux
```PowerShell
echo "ps@notepad" > fichero
foreach ($linea in gc fichero)
{
    $ejecutar = $linea.split("@")[0] + " -name " + $linea.split("@")[1]
    $ejecutar | iex
}
```
