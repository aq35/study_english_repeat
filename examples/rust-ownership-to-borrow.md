# Rust: 所有権を渡すのをやめて、借用にする

> 所有権(ownership)を渡すのをやめて、借用(borrow)にする

scene: GitHub の Rust コードレビュー

## 1. レビュー文(共通のテンプレート)

JA(元)
```
XXXに修正した理由を教えてください。
また、PRの本文または、設計書に、修正した意図を反映してください。
```

JA(推敲)
```
XXXに変更した理由を、このスレッドで教えてもらえますか？
あわせて、その意図をPRの説明文か設計書に記載してください。
```

EN
```
Could you explain why you changed XXX to YYY?
Please also document the reasoning in the PR description or the design doc.
```

EN(柔らかい版)
```
I'd like to understand why this was changed to YYY. Could you also add the intent to the PR description or the design doc?
```

リンキング
```
クッジュ イクスプレイン ワイ ユ チェインジド XXX トゥ YYY
プリーズ オールソウ ダキュメント ザ リーズニンギンザ ピーアール ディスクリプショナーザ ディザインダック
```

連結ポイント
- Could you → クッジュ(d+y が同化して ジュ)
- reasoning in the → リーズニンギンザ(ng+in が連結)
- description or the → ディスクリプショナーザ(n+or が連結)
- design doc → ディザインダック(n+d は軽く繋ぐ)

## 2. コード例(before / after)

### 2-1. `Vec<T>` を `&[T]` に変更

```rust
// before
fn sum(values: Vec<i32>) -> i32 {
    values.iter().sum()
}
let v = vec![1, 2, 3];
let total = sum(v);       // v is moved here
// println!("{:?}", v);   // compile error: v was moved

// after
fn sum(values: &[i32]) -> i32 {
    values.iter().sum()
}
let v = vec![1, 2, 3];
let total = sum(&v);      // v is only borrowed
println!("{:?}", v);      // OK
```

```
Could you explain why you changed `Vec<i32>` to `&[i32]`?
Please also document the reasoning in the PR description or the design doc.
```

### 2-2. `String` を `&str` に変更

```rust
// before
fn greet(name: String) -> String {
    format!("Hello, {name}")
}
greet("Alice".to_string());   // needs an extra allocation

// after
fn greet(name: &str) -> String {
    format!("Hello, {name}")
}
greet("Alice");               // string literal works as is
greet(&owned_string);         // &String is accepted too
```

```
Could you explain why you changed `String` to `&str`?
Please also document the reasoning in the PR description or the design doc.
```

### 2-3. `clone()` を参照渡しに変更

```rust
// before
let total = calc_total(items.clone());   // copies all the data

// after
let total = calc_total(&items);          // no copy, just a borrow
```

```
Could you explain why you changed `items.clone()` to `&items`?
Please also document the reasoning in the PR description or the design doc.
```

## 3. Rust の説明

3つは同じ考え方: **所有権(ownership)を渡すのをやめて、借用(borrow)にする**。

| 用語 | 意味 |
|---|---|
| 所有権 (ownership) | 値の持ち主は常に1人だけ、というルール |
| ムーブ (move) | 値を関数に渡すと持ち主が移る。呼び出し元はその値を使えなくなる |
| 借用 (borrow) | `&` で一時的に貸す。持ち主は変わらず、貸した後も使える |
| スライス | `&[T]` と `&str` のこと。データの一部または全体を指す参照 |

| 変更 | 理由 |
|---|---|
| `Vec<T>` → `&[T]` | 読むだけなら Vec ごと受け取る必要がない。`&Vec<T>`・配列・一部分のスライスも受け取れる |
| `String` → `&str` | 文字列リテラルも `&String` も渡せる。呼び出し側で `.to_string()` が不要 |
| `clone()` → `&` | `clone()` はメモリ確保と丸ごとコピーが発生する。借用ならコピー不要 |

注意: 関数の中でデータを保存する場合(構造体に入れる、別スレッドに渡すなど)は所有権が必要なので変更できない。
