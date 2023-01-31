# Advance Traits

## Associated types
The use of "Associated types" improves the overall readability of code by moving inner types locally into a trait as output types. For example :
```rust
pub trait CacheableItem: Clone + Default + fmt::Debug + Decodable + Encodable {
  type Address: AsRef<[u8]> + Clone + fmt::Debug + Eq + Hash;
  fn is_null(&self) -> bool;
}
```

Using of `Address` is much more clearer and convenient than `AsRef<[u8]> + Clone + fmt::Debug + Eq + Hash`.

1. 🌟🌟🌟
```rust,editable

struct Container(i32, i32);

// A trait which checks if 2 items are stored inside of container.
// Also retrieves first or last value.
trait Contains {
    // Define generic types here which methods will be able to utilize.
    type A;
    type B;

    fn contains(&self, _: &Self::A, _: &Self::B) -> bool;
    fn first(&self) -> i32;
    fn last(&self) -> i32;
}

impl Contains for Container {
    // Specify what types `A` and `B` are. If the `input` type
    // is `Container(i32, i32)`, the `output` types are determined
    // as `i32` and `i32`.
    type A = i32;
    type B = i32;

    // `&Self::A` and `&Self::B` are also valid here.
    fn contains(&self, number_1: &i32, number_2: &i32) -> bool {
        (&self.0 == number_1) && (&self.1 == number_2)
    }
    // Grab the first number.
    fn first(&self) -> i32 { self.0 }

    // Grab the last number.
    fn last(&self) -> i32 { self.1 }
}

fn difference<C: Contains>(container: &C) -> i32 {
    container.last() - container.first()
}

fn main() {
    let number_1 = 3;
    let number_2 = 10;

    let container = Container(number_1, number_2);

    println!("Does container contain {} and {}: {}",
        &number_1, &number_2,
        container.contains(&number_1, &number_2));
    println!("First number: {}", container.first());
    println!("Last number: {}", container.last());
    
    println!("The difference is: {}", difference(&container));
}
```

## Default Generic Type Parameters
When we use generic type parameters, we can specify a default concrete type for the generic type. This eliminates the need for implementors of the trait to specify a concrete type if the default type works.

2. 🌟🌟
```rust,editable

use std::ops::Sub;

#[derive(Debug, PartialEq)]
struct Point<T> {
    x: T,
    y: T,
}

// FILL in the blank in three ways: two of them use the default generic  parameters, the other one not.
// Notice that the implementation uses the associated type `Output`.
impl <T: Sub<Output=T>> Sub<Point<T>> for Point<T> {
    type Output = Self;

    fn sub(self, other: Self) -> Self::Output {
        Point {
            x: self.x - other.x,
            y: self.y - other.y,
        }
    }
}

fn main() {
    assert_eq!(Point { x: 2, y: 3 } - Point { x: 1, y: 0 },
        Point { x: 1, y: 3 });

    println!("Success!");
}
```

## Fully Qualified Syntax
Nothing in Rust prevents a trait from having a method with the same name as another trait’s method, nor does Rust prevent you from implementing both traits on one type. It’s also possible to implement a method directly on the type with the same name as methods from traits.

When calling methods with the same name, we have to use  Fully Qualified Syntax.

#### Example
```rust,editable
trait UsernameWidget {
    // Get the selected username out of this widget
    fn get(&self) -> String;
}

trait AgeWidget {
    // Get the selected age out of this widget
    fn get(&self) -> u8;
}

// A form with both a UsernameWidget and an AgeWidget.
struct Form {
    username: String,
    age: u8,
}

impl UsernameWidget for Form {
    fn get(&self) -> String {
        self.username.clone()
    }
}

impl AgeWidget for Form {
    fn get(&self) -> u8 {
        self.age
    }
}

fn main() {
    let form = Form{
        username: "rustacean".to_owned(),
        age: 28,
    };

    // If you uncomment this line, you'll get an error saying 
    // "multiple `get` found". Because, after all, there are multiple methods
    // named `get`.
    // println!("{}", form.get());
    
    let username = UsernameWidget::get(&form);
    assert_eq!("rustacean".to_owned(), username);
    let age = AgeWidget::get(&form); // You can also use `<Form as AgeWidget>::get`
    assert_eq!(28, age);

    println!("Success!");
}
```

#### Exercise
3. 🌟🌟
```rust,editable
trait Pilot {
    fn fly(&self) -> String;
}

trait Wizard {
    fn fly(&self) -> String;
}

struct Human;

impl Pilot for Human {
    fn fly(&self) -> String {
        String::from("This is your captain speaking.")
    }
}

impl Wizard for Human {
    fn fly(&self) -> String {
        String::from("Up!")
    }
}

impl Human {
    fn fly(&self) -> String {
        String::from("*waving arms furiously*")
    }
}

fn main() {
    let person = Human;

    assert_eq!(Pilot::fly(&person), "This is your captain speaking.");
    assert_eq!(Wizard::fly(&person), "Up!");

    assert_eq!(Human::fly(&person), "*waving arms furiously*");

    println!("Success!");
}
```

## Supertraits
Sometimes, you might need one trait to use another trait’s functionality( like the "inheritance" in other languages ). In this case, you need to rely on the dependent trait also being implemented. The trait you rely on is a `supertrait` of the trait you’re implementing.

4. 🌟🌟🌟
```rust,editable

trait Person {
    fn name(&self) -> String;
}

// Person is a supertrait of Student.
// Implementing Student requires you to also impl Person.
trait Student: Person {
    fn university(&self) -> String;
}

trait Programmer {
    fn fave_language(&self) -> String;
}

// CompSciStudent (computer science student) is a subtrait of both Programmer 
// and Student. Implementing CompSciStudent requires you to impl both supertraits.
trait CompSciStudent: Programmer + Student {
    fn git_username(&self) -> String;
    fn comp_sci_student_greeting(&self) -> String;
}

fn comp_sci_student_greeting(student: &dyn CompSciStudent) -> String {
    format!(
        "My name is {} and I attend {}. My favorite language is {}. My Git username is {}",
        student.name(),
        student.university(),
        student.fave_language(),
        student.git_username()
    )
}

struct CSStudent {
    name: String,
    university: String,
    fav_language: String,
    git_username: String
}
impl Person for CSStudent {
    fn name(&self) -> String {
        self.name.clone()
    }
}
impl Student for CSStudent {
    fn university(&self) -> String {
        self.university().clone()
    }
}
impl Programmer for CSStudent {
    fn fave_language(&self) -> String {
        self.fave_language().clone()
    }
}


// IMPLEMENT the necessary traits for CSStudent to make the code work
impl CompSciStudent for CSStudent {
    fn comp_sci_student_greeting(&self) -> String {
        self.comp_sci_student_greeting()
    }
    fn git_username(&self) -> String{
        self.git_username()
    }
}

fn main() {
    let student = CSStudent {
        name: "Sunfei".to_string(),
        university: "XXX".to_string(),
        fav_language: "Rust".to_string(),
        git_username: "sunface".to_string()
    };

    // FILL in the blank
    println!("{}", comp_sci_student_greeting(&student));
}
```

## Orphan Rules
We can’t implement external traits on external types. For example, we can’t implement the `Display` trait on `Vec<T>` within our own crate, because `Display` and `Vec<T>` are defined in the standard library and aren’t local to our crate. 

This restriction is often called the orphan rule, so named because the parent type is not present. This rule ensures that other people’s code can’t break your code and vice versa. 

It’s possible to get around this restriction using the newtype pattern, which involves creating a new type in a tuple struct.

5. 🌟🌟
```rust,editable
use std::fmt;

// DEFINE a newtype `Pretty` here
struct Pretty(String);


impl fmt::Display for Pretty {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "\"{}\"", self.0.clone() + ", world")
    }
}

fn main() {
    let w = Pretty("hello".to_string());
    println!("w = {}", w);
}
```

> You can find the solutions [here](https://github.com/sunface/rust-by-practice)(under the solutions path), but only use it when you need it :)