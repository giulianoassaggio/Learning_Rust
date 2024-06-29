* rustc = rust compiler
* convenzione snake anche nome file
* rustfmt = rust formatter
* convenzione indentazione 4 spazi
# Cargo
Cargo is Rust’s build system and package manager.
* `cargo new` = creating new proj
* toml = rust configuration language
* dependencies: package of code are called crates	
* cargo si aspetta che il codice stia dentro la cartella  src
* per compilare non si usa tendenzialmente rustc direttamente, ma cargo: `cargo build`. di default fa la build di debug
* `cargo run` per runnare. In realtà compila e runna insieme. se già compilato obv non fa di nuovo
* `cargo check` controlla se il codice è compilabile
* per buildare in release, `cargo build --release`. al posto di `target/debug`, salverà in `target/release`
* cargo ensures a reproducible build usando il file Cargo.lock. 
* Se invece voglio aggiornare un crate, ho il comando `update`, che ignora cargo lock e poi lo modifica con la versione compatibile più recente
# Guessing game
```rs
use std::io;

fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {}", guess);
}
```
* use std::io -> sto portando il modulo di io, contenuto nella standard library, nel mio scope.
* fn -> declaration of a new function. parentesi come C. println! macro (più avanti)
* `let` crea la variabile, di default immutabile dopo il binding. per permettere l'assegnamento, va esplicitamente aggiunto il `mut`
* `String::new` -> new è una funzione associata al tipo Stringa. 
* `io::stdin()` sto chiamando la funzione stdin definita nella libreria io. Essa ritorna un'istanza del TIPO Stdin, che ha associati dei metodi, tra cui per esempio `.read_line(&mut guess)`. & è una reference (anch'esse immutable by def)
* `.expect(...)` è un metodo associato al tipo Result (tipo di ritorno ddella funzione read_line). Result è un enum, ma ne parleremo + avanti. Comunque, Result contiene due variants: Ok e Err. Se Result è un Err value, expect farà crashare il programma mostrando il mex d'errore. If this instance of Result is an Ok value, expect will take the return value that Ok is holding and return just that value to you so you can use it. In this case, that value is the number of bytes in the user’s input. NOTARE CHE il modo giusto di gestire gli errori è scrivere effettivamente codice per gestirli, ma se vogliamo semplicemente il crash, expect va bene.
* `println!("You guessed: {}", guess);` abbastanza intuitivo l'uso dei placeholder. Quando si printa il valore della variabile, essa può andare dentro le parentesi, se invece va computata un'espressione, va messa per forza dopo la virgola

## generating secret number
Rust non include random number functionality nella standar lib, ma ha un "crate", rand, con queste funzionalità
Per includere questo crate, va modificato il TOML, sotto dependencies.

```rust
use std::io;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    println!("The secret number is: {secret_number}");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {guess}");
}
```
* `rand::Rng` definisce metodi che implementati dal rng (random number gerator)
* `rand::thread_rng` fornsce il seed
* `gen_range`: interessante l'espressione che prende in input: è un range nella forma start..=end, include entrambi gli estremi.

## Comparin guess e secret
Now that we have user input and a random number, we can compare them. That step is shown in Listing 2-4. Note that this code won’t compile just yet, as we will explain.

```rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    // [...]

    println!("You guessed: {guess}");

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
}
```
* Ordering è un altro enum, ha tre valori: Less, Greater, Equal
* cmp è un metodo chiamabile su qualsiasi oggetto comparabile, e ritorna un Ordering.
* `match`: ci permette di decidere cosa fare in base al valore di ritorno. Un match è fatto da "arms". un "arm" cosiste in un pattern da matchare e dal codice che deve essere runnato di conseguenza

Perché non compila? rust has a strong, static type system. However, it also has type inference. Quando abbiamo dichiarato la stringa guess, rust ha capito da solo il tipo, senza che servisse esplicitarlo. Idem per il numero. per default gli interi sono i32. Ad ogni modo, non compila perché nn è definita la comparazione tra string e i32. 
```rust
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```
Nota interessante: rust ci permette di shadoware il nome di una variabile, e chiamarne un'altra con lo stesso nome, molto utile se voglio solo convertire i tipi senza dover inventarmi ogni volta due nomi diversi.
```rust
guess.trim().parse()
```
Il guess di questa espressione si riferisce alla stringa originale, su cui è hiamato il meotdo trim (elimina spazi a inizio e fine), e poi parso a intero. Perché intero? lo abbiamo suggerito a rust con :u32
ecause it might fail, the parse method returns a Result type, quindi usiamo expect come prima.
## Adding a loop
banalissimo
```rust
loop {
	...
 	...
	match ... {
		Ordering::Equal => {
			println!("Guessed");
			break;
		}
	}
}
```
## Handling invalid inputs

```rust 
        // --snip--

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {guess}");

        // --snip--

```
siamo passati da una chiamata di expect a una espressione match che ci permette di gestire l'enum di Result. Infatti, se il parse non va a buon fine, va a matchare Err(_) nel secondo braccio, l'underscore e un catchall value: non ci interessa cosa ci sia effettivamente dentro. il programma continuerà il loop, ignorando l'input invalido

# Common Programming Concepts
