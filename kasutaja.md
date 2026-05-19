## SQL Server – Kasutajate autentimine ja õiguste haldamine
Mis on autentimine SQL Serveris?
 ### Autentimine tähendab kasutaja tuvastamist ehk kontrollimist, kas kasutajal on õigus SQL Serverisse sisse logida.

**SQL Serveris kasutatakse kahte peamist autentimise tüüpi:**

1. Windows Authentication
Selle puhul kasutatakse samu kasutajaandmeid, millega logitakse sisse Windows operatsioonisüsteemi.

>Kasutajanimi ja parool on seotud Windowsiga. 
>Turvalisem lahendus. 
>Paroole haldab Windows. 
>Kasutaja ei pea eraldi SQL Serveri parooli teadma.

2. SQL Server Authentication
>Selle puhul luuakse kasutaja otse SQL Serverisse.
>Kasutaja ei ole seotud Windowsiga. 
>Määratakse eraldi kasutajanimi ja parool. 
>Sobib veebirakenduste jaoks. 
---------------------------------------------------------------
**Näide kasutajast: DirectorIrina. Parool: director**
----------------------------------------------------------------
## Kasutaja loomine SQL Serveris
1. Serveritaseme kasutaja loomine (Login) Sammud Ava:
Security → Logins Tee permklikk ja vali:
New Login...
<img width="703" height="658" alt="{91720B7D-EB47-439C-B814-DE89003DC1F7}" src="https://github.com/user-attachments/assets/d8ed57ac-e049-43b9-9e68-0b4cfda6a055" />

Harjutamiseks võib eemaldada linnukese:  User must change password at next login.

**Server Roles**
Menüüst Server Roles saab määrata serveri üldised õigused.

Tavaliselt piisab rollist: public

<img width="700" height="656" alt="{F56F1B14-EACD-4D5E-A50C-BE9FBDD5C94B}" src="https://github.com/user-attachments/assets/47d95f79-a8cf-44a3-8cfc-349a62c8d25d" />
<img width="700" height="654" alt="{EF9FA9ED-D06C-467A-8AE7-586130422010}" src="https://github.com/user-attachments/assets/8d6db437-358b-4671-ba19-36b050c4b634" />

```sql
--GRANT - õiguste määramine
--DENY - õiguste keelamine
-- anname kasutajale õigus vaadata tabelit (SELECT), 
-- lisada admeid (INSERT) ning uuendada neid(UPDATE)
GRANT SELECT ON opilane TO direktorRuslan;
GRANT INSERT ON opilane TO direktorRuslan;
GRANT UPDATE ON opilane TO direktorRuslan;

DENY DELETE ON opilane TO direktorRuslan;
```
## Kasutaja õiguste kontroll

1. tuleb sisselogida kasutajana directorIrina. Connect--> Database Engine

<img width="474" height="510" alt="{3CB9D837-5AB8-434C-8110-A27A0EB28F3F}" src="https://github.com/user-attachments/assets/6c685b9e-e878-4048-8ae1-749f27c1063e" />

2. saab tabeli sisu näha ja sisestada uus kiri.
<img width="711" height="595" alt="{58C6D6C3-024C-404F-B3DA-DCE790117D89}" src="https://github.com/user-attachments/assets/73cfa3ed-ea2d-487c-9c4e-beb0980db9d4" />

3. saab lisada uusi andmeid.
<img width="621" height="583" alt="{98E1CD55-B1E1-45E8-B8CF-4C4C8736C53E}" src="https://github.com/user-attachments/assets/db4c0fd9-18b3-409f-a502-7f4781beb283" />

4. kontrollime tegevus, mis ei ole lubatud kasutajale, näiteks andmete kustutamine
<img width="1220" height="586" alt="{30A42E5F-3CEC-4031-81FD-72CF0DBDC42B}" src="https://github.com/user-attachments/assets/0cd87ebf-e98a-4f72-bb8d-1e4a303ee68a" />

5. Selle käsuga saate kontrollida, millised õigused meie kasutajal on.
<img width="851" height="718" alt="{3A7ED3BF-E797-4D0B-982E-3025399C3E43}" src="https://github.com/user-attachments/assets/505950e1-b7cf-4130-b9fa-6cc3838f952f" />


