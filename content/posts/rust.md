+++
title = "Rust: The best Language for Programmers with ADHD"
date = "2025-01-16"

[taxonomies]
tags=["Rust","ADHD","coding"]
+++

## <span style="color:orange;">My Story with Different Programming Languages</span>

I've worked with many programming languages over the years. In C, I often made mistakes with memory. I would forget to free memory or use pointers wrongly. In TypeScript, even though it helped me with types, I still made mistakes when I lost focus.

## <span style="color:orange;">How Rust Helps Me Code Better</span>

When I started using Rust, it felt like having a helpful friend always sitting by my side who stopped me from making mistakes.
The Rust compiler is like a friend who checks your work. It tells you exactly what's wrong and how to fix it. This is perfect when your mind sometimes skips important details. 

People with ADHD often have a **"now or never"** brain - we either hyperfocus or struggle to focus at all. This makes it easy to skip important details when coding. I used to write code quickly when I was in the flow, but would forget about error handling or memory management.

### <span style="color:orange;">The Compiler Watches Out for You</span>

```rust
fn main() {
    let name = String::from("Enes");
    let greeting = name;  // name moves here
    println!("Hello, {}", name);  // Error: name is not valid anymore!
}
```

Think about how our ADHD brain works: we often have multiple thoughts racing through our mind, like browser tabs open in the background. In other languages, this can lead to:
- Forgetting we already used a variable
- Not remembering if we freed memory
- Missing important error checks because we got distracted
- Accidentally using null values because we forgot to initialize something

But Rust's compiler is like having a supportive friend who:
1. Catches our mistakes before they become problems
2. Gives us clear, step-by-step instructions to fix issues
3. Forces us to slow down and think about data ownership
4. Provides immediate feedback, which is crucial for ADHD minds

But more importantly, Rust forces us to write **type-safe code**. This is a game-changer for the ADHD brain. Let me show you what I mean:
```rust
// In other languages, we might write code like this: (at least I did :D)
// let result = someFunction();  // What type is result? Who knows! :(

// In Rust, we must be clear about our types:
enum TaskStatus {
    InProgress,
    Done(String),
    Failed(Error)
}

fn process_task(task: Task) -> TaskStatus {
    // Rust won't let us forget to handle all possible cases
    match task.status {
        TaskStatus::InProgress => {
            // We must handle this case
        },
        TaskStatus::Done(message) => {
            // We must handle this case
        },
        TaskStatus::Failed(error) => {
            // We must handle this case
        }
    }
}
```

This type safety is like having **guard rails for our racing thoughts**. When our mind wants to skip ahead, Rust gently pulls us back. It turns our ADHD tendency to miss edge cases into a structured process of handling each possibility.

So the beauty of Rust isn't just that it catches our mistakes - it teaches us to think in a more organized way. Each variable must have a clear type. Each function must specify what it returns. Each error must be handled. For the ADHD brain that often struggles with organization, this external structure becomes a **powerful tool** for writing better code.

## <span style="color:orange;">The Hard Parts</span>

Let's be honest - Rust isn't always easy, especially for brains that want everything to happen right now:

- **Takes Time to Learn**: Learning about ownership and borrowing can be tough at first. When your mind races ahead, wanting to build cool things, it's frustrating to deal with these new concepts.

- **Sometimes Too Strict**: The compiler can make simple tasks feel complicated. You might think "I just want to share this data between two functions, why is it so hard?!"

- **Lots of Documentation**: You need to read docs often, which can break your focus. For ADHD folks who might struggle with context switching, having to pause and read documentation can feel like slamming the brakes on your thought process. Also, creates in Rust are not mature as in other languages. They always change and you need to update your codebase frequently and read the documentation (and in most cases they dont have documatation but just code examples :D)

- **Fighting with the Borrow Checker**: Sometimes you just want to write code quickly, and the borrow checker feels like that friend who keeps interrupting your story to correct small details.

yes, it's annoying to put on every time, but it saves us from bigger problems later.

Remember: **"This five-minute battle with the compiler now might save me five hours of debugging later."** :-)