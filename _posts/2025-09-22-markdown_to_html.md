---
layout: post
author: Abhinav Saxena
tags: [overview, moonwalk]
title: "Markdown to HTML"
date: 2025-09-22
---

## Rust로 만든 간단한 markdown -> html 변환기

### 폴더 구조
src/<br/>
├─ main.rs        // CLI 처리, 파일 입출력. <br/>
├─ parser.rs      // Markdown 파싱 모듈. <br/>
├─ transformer.rs // Markdown 요소를 HTML 문자열로 변환.<br/> 
└─ codegen.rs     // HTML 문서 전체 구조 생성 및 출력. 
<br/>
### 실행 방법
```bash
cargo run input.md output.html
```
<br> 


### 실행 흐름

Markdown 파일 읽기 → parser.rs로 파싱 → `Vec<MarkdownElement>` 생성 → 
transformer.rs에서 HTML 블록 변환 → codegen.rs에서 최종 HTML 구조 생성 → 파일 출력

<br>

main.rs
```rust
// ============= 안승우 파트 =============

mod parser; //parser.rs 가져오기
mod transformer; //transformer.rs 가져오기
mod codegen; //codegen.rs 가져오기

use std::env; //명령값 인자 받을 떄 사용
use std::fs; // 파일 읽고 쓸때 사용
use std::process; //프로그램 종료할때 사용

fn main() {
    let args: Vec<String> = env::args().collect(); //명령값 인자 다 받아서 Vec에 저장

    if args.len() < 3 { //args의 길이가 3 미만이면 (명령값 인자가 부족하면)
        eprintln!("사용법 : {} <yourfile.md> <output.html>", args[0]); //사용법 출력
        process::exit(1); //프로그램 종료 코드 1 반환
    }

    let mdfile = &args[1]; //md 파일 저장
    let htmlfile = &args[2]; //html 파일 저장

    let md = match fs::read_to_string(mdfile) { //mdfile 읽어서 md에 저장
        Ok(text) => text, //쭉 흘러가면 그 안에 내용 저장
        Err(err) => { // 에러가 나면
            eprintln!("입력 파일 읽기 오류!! : {}", err); //입력 파일 읽기 오류와 에러 내용 출력
            process::exit(1); //프로그램 종료 코드 1 반환
        }
    };

    let elements = parser::do_parse(&md); //파싱
    let html = codegen::generate_html(elements); //html 코드 변환

    if let Err(err) = fs::write(htmlfile, html) {
        eprintln!("출력 파일 저장 오류 : {}", err);
        process::exit(1);
    }

    println!("컴파일 성공 : {} -> {}", mdfile, htmlfile);
}
```

`env::args()` : 프로그램 실행 시 전달된 모든 CLI 인자를 수집
`args.len() < 3` 조건 : 프로그램 이름, 입력 파일, 출력 파일 세 개가 필요하므로 부족하면 종료

`fs::read_to_string(mdfile)` : 파일 내용을 문자열로 읽기

`fs::write()` : 변환된 HTML 문자열을 지정한 파일에 작성
<br/>
<br/>

parser.rs
```rust
// ============= 안승우 파트 =============

//내용 설명 : 마크 다운 파일에 일반 단락과 빈칸 그리고 헤더를 나눠줌 (줄마다) 헤더는 #의 갯수에 따라 h1, h2, h3로 나눠줌

// ============= 마크 다운의 3요소 =============
#[derive(Debug, Clone)]
pub enum MarkdownElement {
    Header{ count: u8, text: String }, // h1 = count 1, h2 = count 2, h3 = count 3
    Ptag(String), //p태그
    BlankLine, //빈칸
}

// ============= 마크 다운 파싱 함수 =============
pub fn do_parse(md: &str) -> Vec<MarkdownElement> {
    let mut elements = Vec::new();

    for line in md.lines() {
        let trimmed_line = line.trim();

        if trimmed_line.is_empty() {
            elements.push(MarkdownElement::BlankLine);
        } else if trimmed_line.starts_with("###"){
            elements.push(MarkdownElement::Header{
                count: 3,
                text: trimmed_line[4..].to_string()
            })
        } else if trimmed_line.starts_with("##"){
            elements.push(MarkdownElement::Header{
                count: 2,
                text: trimmed_line[3..].to_string()
            })
        } else if trimmed_line.starts_with("#"){
            elements.push(MarkdownElement::Header{
                count: 1,
                text: trimmed_line[2..].to_string()
            })
        } else {
            elements.push(MarkdownElement::Ptag(trimmed_line.to_string()));
        }
    }

    elements
}

// ============= 출력 예시 =============
// ============= example.md =============
// # Header 1
// This is a paragraph.
// ## Header 2
// Another paragraph.
// ### Header 3
// ============= 출력 =============
// [Header { count: 1, text: "Header 1" },
//  Ptag("This is a paragraph."),
//  Header { count: 2, text: "Header 2" },
//  Ptag("Another paragraph."),
//  Header { count: 3, text: "Header 3" }]
```
`MarkdownElement` : AST와 비슷한 역할을 하는 enum
`### 제목` → `Header { count: 3, text: "제목" }`형태로 변환에 필요한 정보들을 담고있음

`for line in md.lines()` :  문자열을 줄 단위로 순회
모두 순회 후 `Vec<MarkdownElement>`를 반환한다.

<br>

transformer.rs
```rust
// ============= 이재훈 파트 =============

use regex::Regex;
use crate::parser::MarkdownElement;

// MarkdownElement 하나를 HTML 문자열로 변환
pub fn transform_element(element: &MarkdownElement) -> String {
    match element {
        MarkdownElement::Header { count, text } => {
            // inline Markdown 처리
            let content = transform_inline(text);
            // h1 ~ h3 태그로 변환
            format!("<h{count}>{content}</h{count}", count = count, content = content)
        }
        MarkdownElement::Ptag(text) => {
            // 문단 텍스트를 <p>로 감싸고 inline 처리
            let content = transform_inline(text);
            format!("<p>{}</p>", content)
        }
        MarkdownElement::BlankLine => "<br/>".to_string(), // 빈 줄은 <br>로 변환
    }
}

// inline Markdown 포맷 처리 함수
fn transform_inline(text: &str) -> String {
    let mut s = text.to_string();

    // HTML 이스케이프 문자 처리
    s = s.replace("&", "&amp;").replace("<", "&lt;").replace(">", "&gt;");

    // inline code `code` → <code>code</code>
    let re_code = Regex::new(r"`([^`]+)`").unwrap();
    s = re_code.replace_all(&s, "<code>$1</code>").to_string();

    // bold **text** → <strong>text</strong>
    let re_bold = Regex::new(r"\*\*(.+?)\*\*").unwrap();
    s = re_bold.replace_all(&s, "<strong>$1</strong>").to_string();

    // italic *text* → <em>text</em>
    let re_italic = Regex::new(r"\*(.+?)\*").unwrap();
    s = re_italic.replace_all(&s, "<em>$1</em>").to_string();

    // link [text](url) → <a href="url">text</a>
    let re_link = Regex::new(r"\[([^\]]+)\]\(([^)]+)\)").unwrap();
    s = re_link.replace_all(&s, "<a href=\"$2\">$1</a>").to_string();

    s
}
```

`regex::Regex` : 정규표현식을 사용해 inline Markdown 포맷(`**bold**`, `*italic*`, `` `code` ``, `[link](url)`)을 HTML로 변환
<br>
```rust
let re_code = Regex::new(r"`([^`]+)`").unwrap();
s = re_code.replace_all(&s, "<code>$1</code>").to_string();
```
`` `code` `` 패턴을 `<code>code</code>`로 변환
정규식 `` `([^`]+)` `` : 백틱 내부의 문자열을 캡처

모든 inline 변환을 적용한 문자열 `s` 반환



codegen.rs
```rust
// ============= 이재훈 파트 =============

use crate::parser::MarkdownElement;
use crate::transformer;

// MarkdownElement 리스트를 HTML 문자열로 변환
pub fn generate_html(element: Vec<MarkdownElement>) -> String {
    let mut html = String::new();

    // HTML 문서 기본 구조 작성
    html.push_str("<!DOCTYPE html>\n<html>\n<head>\n<meta charset=\"UTF-8\">\n<title>Markdown</title>\n</head>\n<body>\n");

    // Markdown AST 리스트를 순회하며 각 요소를 HTML로 변환
    for elem in element {
        html.push_str(&transformer::transform_element(&elem)); // 블록 단위 HTML 변환
        html.push_str("\n"); // 줄바꿈 추가
    }

    // HTML 문서 닫기
    html.push_str("</body>\n</html>\n");
    
    html
}
```

`Vec<MarkdownElement>`를 전체 HTML 문서로 변환


완성된 HTML 문자열을 반환 후 `main.rs`에서 파일로 저장한다.