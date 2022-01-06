# Vocabulary

Our major targets learn English as foreign language, mostly teenagers so **Khala** language keywords should be restricted to beginner to pre-intermediate vocabulary, and favor daily English over technical jargons.

`Maybe<T>` is better than `Option<T>`.

`eudplib` and `epScript` support non-ASCII identifier (thanks to Python), so **Khala** must support too.

## Unsafe keyword

My alterative is `careful`, very intuitive naming for notifying what users to be. `unsafe` is not daily term; I would say dangerous or "Watch out!" instead. I think Rust `unsafe` keyword is implementor friendly naming. `unsafe` has two semantics; `unsafe fn` alerts users to read doc before use and *must* ensure no invariant is allowed, `unsafe { }` means the writer manually had proved its soundness or should. Either way it does not mean the code itself is unsafe.

Although C#, Rust, Swift, Go use `unsafe`, our targets don't have previous programming experience so familiarity to other languages gain little. Also `unsafe` is too scary name for beginners.

I dislike long name like `i_assert_that_this_code_is_safe`. It's bad name for tooling against out of bound, null reference, game crash, desync, unsupported EUD errors etc. `unsafe` usages in **Khala** would be no less than Rust's.

## Further reading

[Rename `unsafe` to `trusted`][rename_unsafe]

[rename_unsafe]: https://github.com/rust-lang/rfcs/pull/117
