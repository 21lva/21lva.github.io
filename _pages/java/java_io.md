---
layout: post
title:  "IO"
date:   2019-12-18 01:02:59
author: Inhyuk
name: java_io.md
---

입출력 스트림
======

- 두 대상 사이에 데이터를 전달하려면 **stream** 이 있어야 함
- **stream** : 데이터를 운반하는 통로. FIFO구조
  - 입력 스트림
  - 출력 스트림

- - -

바이트 스트림
============

- InputStream, OutputStream의 자손들. 각각 읽고 쓰는데 필요한 추상메서드를 구현
- InputStream : FileInputStream,ByteArrayInputStream, PipedInputStream, AudioInputstream
  - abstract int read() : 1byte를 읽어옴. 읽을 데이터가 없으면 -1 리턴.
  - int read(byte[] b) : 배열 b의 크기만큼 읽어서 배열에 저장. 읽어온 데이터 수 리턴
  - int read(byte[] b,int off, int len) : 최대 len개의 byte를 읽어서 배열 b에 off부터 저장한다
  - void close() : 스트림을 닫고 자원을 반납
  - void mark(int readlimit) : 현재위치 표시. 후에 reset()에 의해서 표시해 놓은 위치로 다시 돌아감. readlimit= 되돌아 갈 수 있는 byte 수
  - boolean markSupported : mark()와 rest()을 지원하는지 여부
  - void reset() : 스트림의 위치를 mark이 호출되었던 위치로 되돌림
  - long skip(long n) : 주어진 n만큼 건너뜀

- OutputStream : FileOutputStream,ByteArrayOutputStream, PipedOutputStream, AudioOutputstream
  - abstract void write(int b) : 주어진 값을 출력 소스에 write
  - void write(byte[] b) : 주어진 배열 b에 저장된 내용을 write
  - void write(byte[] b,int off, int len) : 주어진 배열 b의 off부터 len개 만큼 write
  - void close() : 스트림 종료. 자원 반납
  - void flush() : 스트림의 버퍼에 있는 모든 내용을 write

- 프로그램이 종료되면 JVM이 자동으로 스트림을 끊어버리기는 하지만 close하는 습관을 들이자

ByteArrayInputStream, ByteArrayOutputStream
------------------------------------

- 바이트 배열에 데이터를 입출력 할 때 사용
- 다른 곳에 입출력하기 전에 데이터를 임시로 바이트 배열에 담아서 변환할 때 사용
- 자주 사용하지는 않음 ㅎ

```java
byte[] inSrc={0,1,2,3,4,5,6,7};
byte[] outSrc = null;

ByteArrayInputStream input = new ByteArrayInputStream(inSrc);
ByteArrayOutputStream output = new ByteArrayOutputStream();

int data=0;
while((data=input.read())!=-1){
  output.write(data);
}

outSrc = output.toByteArray();

//read, write를 연속으로 진행할 수 있다
byte[] tmp = new byte[10];
input.read(tmp,0,tmp.length);
output.write(tmp,5,5);

outSrc = output.toByteArray();
```

FileInputStream, FileOutputStream
---------------------------------

- 파일 입출력을 위한 스트림
- 실제로 많이 사용
- 매개변수로 String 객체와, File 객체, FileDescriptor 객체를 받을 수 있다
- FileOutputStream은 append(boolean) 을 받아서 이어쓰기인지, 덮어쓰기를 결정
- 포함하는 메서드에 IOException을 적어야 한다

```java
FileInputStream fis = new FileInputStream("read.txt");
FileOutputStream fos = new FileOutputStream("write.txt");
int data = 0;

while((data=fis.read())!=-1){
  fos.write(data);
}
fis.close();
fos.close();
```


- - -

보조 스트림
==========

- 실제 데이터를 주고 받는 스트림이 아니라 스트림의 기능을 향상시키거나 새로운 기능을 추가
- 보조 스트림만으로 입출력 불가능. 스트림을 생성한 후에 보조 스트림을 생성해야 함
- FilterInputStream, FilterOutputStream : 필터를 이용한 입출력 처리
- BufferedInputStream, BufferedOutputStream : 버퍼를 이용한 입출력 성능 향상
- DataInputStream, DataOutputStream : int, flot와 같은 primitive type로 데이터를 처리하는 기능
- SequenceInputStream : 두 개의 스트림을 하나로 연결
- LineNumberInputStream : 읽어온 데이터의 라인 번호를 카운트
- ObjectInputStream, ObjectOutputStream : 데이터를 객체 단위로 읽고 쓰는데 사용
- PrintStream : 버퍼를 이용, 추가적인 printf 관련 기능
- PushbackInputStream : 버퍼를 이용해서 읽어 온 데이터를 다시 되돌리는 기능

```java
//파일 스트림 생성
FileInputStream fis = new FileInputStream("abc.text");
//보조 스트림 생성. 버퍼 제공
BufferedInputStream bis = new BufferedInputStream(fis);

bis.read();
```

바이트 기반 보조 스트림
--------------------

#### 1 FilterInputStream, FilterOutputStream

- InputStream, OutputStream의 자손. 모든 보조스트림의 조상
- 자체로는 아무런 일도 하지 않고, 상속을 해서 메서드를 오버라이딩 해야함
- Buffered[]Stream, Data[]Stream, Pushback[]Stream이 상속함

#### 2. BufferedInputStream, BufferedOutputStream

- 스트림의 효율을 높이기 위해 버퍼를 사용하는 보조 스트림
- 한 byted씩 입출력 하는 것 보다 버퍼(바이트 배열)을 이용해서 한 번에 여러개의 바이트를 입출력해서 속도가 빠름
- 생성자
  - BufferedInputStream(InputStream in, int size) : 지정된 크기(size, 기본값=8192)의 버퍼를 가지는 스트림 생성
  - BufferedOutputStream(InputStream in, int size) : 지정된 크기(size, 기본값=8192)의 버퍼를 가지는 스트림 생성
  - flush() : 버퍼의 모든 내용을 출력소스에 출력한 후에 버퍼를 비움.
  - close() : flush()를 호출한 후에 연결 끊음

```java
//생성
BufferedInputStream bis = new BufferedInputStream(new FileInputStream("read.txt"),5);
BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("write.txt"),5);

//읽고 저장
byte[] receive = new byte[5];
int readCnt=0;
while((readCnt=bis.read(receive,0,receive.length))!=-1){
    bos.write(receive,0,readCnt);//읽은 만큼 쓰지 않으면 마지막에 3개를 읽었지만 앞에 있던 결과(receive[3],receive[4])가 쓰일 수도 있음
}
```

#### 3. DataInputStream, DataOutputStream

- Filter[]Stream의 자손. Data[] 인터페이스 구현
- byte단위가 아닌 8가지 기본 자료형의 단위로 읽고 씀

- DataInputStream의 생성자와 메서드
  - DataInputStream(InputStream in)
  - boolean readBoolean()
  - byte readByte()
  - char readChar()
  - short readShort()
  - int readInt()
  - long readLong()
  - float readFloat()
  - double readDouble()
  - int readUnsignedByte()
  - String readUTF()
  - static String readUTF(DataInput in) : 입력 스트림(in)에서 UTF-8형식의 유니코드를 읽음
  - int skipByte(int n) : n 만큼 건너뜀


- DataOutputStream(OutputStream out)
  - void writeBoolean()
  - void writeByte()
  - void writeChar()
  - void writeShort()
  - void writeInt()
  - void writeLong()
  - void writeFloat()
  - void writeDouble()
  - void writeUnsignedByte()
  - void writeUTF()
  - void writeChars(String s) : 주어진 문자열을 출력. writeChar(int c)를 여러번 출력한 것과 같은 것
  - int size() : 지금까지 DataOutputStream에 쓰여진 byte의 수

#### 4. SequenceInputStream

- 여러 개의 입력 스트림을 연속적으로 연결해서 하나의 스트림으로부터 데이터를 읽는 것처럼 처리할 수 있도록 도와줌
- 큰 파일을 여러 개의 작은 파일로 나누었다가 하나의 파일로 합치는 것과 같은 작업을 수행할 때 사용

- - -

문자기반 스트림
============

- java에서는 문자인 char이 1byte가 아닌 2byte -> 바이트 기반으로 2byte 문자를 처리하는 것이 불편
- InputStream -> Reader, OutputStream -> Write
- 보조 스트림도 문자기반 보조 스트림 존재

Reader, Writer
-------------

- 문자기반 스트림의 조상 클래스
- byte대신 char을 사용하는 것외에는 InputStream/OutputStream과 다른 것이 없음
- Reader
  - abstract int read(char[] c, int off, int len) : 주어진 갯수 만큼 c로 읽어옴. 읽을 데이터가 없으면 -1 리턴.
  - int read(char[] c) : 입력 소스로부터 c 크기만큼 읽어서 저장. 읽을 것이 없으면 -1 리턴
  - int read() : 문자 하나(char) 읽어서 반환. 읽을 것이 없으면 -1 리턴
  - int read(CharBuffer target) : 입력소스로부터 읽어서 문자버퍼에 저장
  - boolean ready() : 데이터 읽을 준비가 되었는지 확인
  - void close() : 스트림을 닫고 자원을 반납
  - void mark(int readlimit) : 현재위치 표시. 후에 reset()에 의해서 표시해 놓은 위치로 다시 돌아감. readlimit= 되돌아 갈 수 있는 byte 수
  - boolean markSupported : mark()와 rest()을 지원하는지 여부
  - void reset() : 스트림의 위치를 mark이 호출되었던 위치로 되돌림
  - long skip(long n) : 주어진 n만큼 건너뜀

- Writer
  - Writer append(char c) : 지정된 문자를 출력소스에 저장
  - Writer append(CharSequence c) : 지정된 문자열(CharSequence)를 출력 소스에 출력
  - Writer append(CharSequence c, int start, int end) : 지정된 문자열의 부분을 출력
  - abstract void write(int b) : 주어진 값을 출력 소스에 write
  - void write(int b) : 주어진 값을 출력소스에 저장
  - void write(char[] c) : 주어진 배열 c에 저장된 모든 내용을 출력소스에 저장
  - void write(char[] c,int off, int len) : 주어진 배열 c의 off부터 len개 만큼 write
  - void write(String s) : 주어진 문자열을 출력소스에 저장
  - void write(String s,int off, int len) : 주어진 문자열의 일부를(off부터 len개 만큼) write
  - abstract void close() : 스트림 종료. 자원 반납
  - abstract void flush() : 스트림의 버퍼에 있는 모든 내용을 write

FileReader, FileWriter
-------------------

- 파일로부터 텍스트데이터를 일고, 파일에 쓰는데에 사용

PipedReader, PipedWriter
-----------------------

- 쓰레드 간에 데이터를 주고 받을 때 사용
- 다른 스트림과 달리 입력과 출력 스트림을 하나의 스트림으로 연결해서 데이터를 주고 받음
- 스트림을 생성한 다음에 한쪽의 쓰레드에서 **connect()** 를 호출해서 입력스트림과 출력 스트림을 연결
- 한 쪽만 닫아도 양쪽 모두 닫힘

StringReader, StringWriter
-------------------------

- 입출력 대상인 메모리인 스트림

- - -

문자 기반의 보조 스트림
======================

BufferedReader, BufferedWriter
-----------------------------

- 버퍼를 이용해서 입출력을 함
- 효율이 매우 좋음
- readline을 통해서 데이터를 라인단위로 읽을 수 있음

```java
FileReader fr = new FileReader("reader.txt");
FileWriter fw = new FileWriter("write.txt");
BufferedReader br = new BufferedReader(fr);
BufferedWriter bw = new BufferedWriter(fw);

String rString = "";
while((rString= br.readLine())!=null){
    System.out.println(rString);
    bw.write(rString);
    bw.newLine();
}
bw.close();
br.close();
```

InputStreamReader, OutputStreamWriter
------------------------------------

- 바이트 기반 스트림을 문자기반 스트림으로 연결시켜주는 역할
- 바이트 기반 스트림의 데이터를 지정된 인코딩의 문자데이터로 변환하는 작업 수행
- 생성자와 메서드
  - InputStreamReader(InputStream in, String encoding) : 지정된 인코딩을 사용하는 InputStream 생성. encoding의 기본값은 os마다 다름
  - OutputStreamReader(OutputStream ou, String encoding) : 지정된 인코딩을 사용하는 OutputStream 생성. encoding의 기본값은 os마다 다름
  - String getEncoding() : 지정된 인코딩 알려줌

- - -

표준 입출력
=============

- 콘솔 창을 통한 데이터 입출력
- System.in(InputStream), System.out(PrintStream), System.err(PrintStream) 사용
- 입력을 받을시 '\\r', '\\n'과 같은 문자가 붙어오고 이를 제거해야하는 불편함이 존재
- 나중에 나올 **Scanner** 와 **Console** 클래스를 사용하자

setOut,setErr,setIn
------------------

- 입출력 대상을 콘솔에서 다른 스트림으로 변경
- setOut(PrintStream out) : System.out의 출력을 지정된 PrintStream으로 변경
- setErr(PrintStream err) : System.err출력을 지정된 PrintStream으로 변경
- setIn(InputStream in) : System.in 입력을 지정된 InputStream으로 변경

Scanner
----------

- java.utill 패키지에 포함되어 있음
- 메서드
  - String next() : 문자열을 입력 받음. 공백을 기준으로 짜름
  - String nextLine() : 문자열을 입력 받음. 개행을 기준으로 짜름
  - int nextInt() : int를 입력 받음
  - long nextLong() : long을 받음
  - double nextDouble() : double을 받음
  - float nextFloat() : float를 받음

```java
//Scanner 객체 생성
Scanner sc = new Scanner(System.in);

//입력
String s1 = sc.next();
String s2 = sc.nextLine();
int i = sc.nextInt();
long l = sc.nextLong();
float f = sc.nextFloat();
double d = sc.nextDouble();
```

- 위의 순서대로 하면 제대로 되지 않는다. next()를 수행하면 개행문자가 남아서 오류가 생긴다. nextLine이전에 nextLine을 한번 해서 개행을 제거해야 한다

- - -

File
=====

- 파일과 디렉토리 관련 메서드가 있는 클래스
- [API 문서](https://docs.oracle.com/en/java/javase/13/docs/api/java.base/java/io/File.html)
- [참고자료](https://hyeonstorage.tistory.com/233)

생성자
------

- File(String fileName) : 주어진 문자열을 이름으로 갖는 파일을 위한 File 인스턴스 생성. 경로를 지정해서 해줘도 됨
- File(String pathName,String filename), File(File pathName,String filename) : 파일 경로와 이름을 따로 분리하여 지정
- File(URL uri) : 지정된 uri로 파일 생성

메서드
-----

- String getName() : 파일이름을 리턴
- String getPath() : 파일 경로를 리턴
- String getAbsolutePath() : 파일의 절대경로 리턴
- File getAbsoluteFile() : 파일의 절대경로를 File 객체로 리턴
- String getParent() : 파일의 조상 디렉토리를 String으로 리턴
- File getParentFile() : 파일의 조상 디렉토리를 File객체로 리턴
- String getCanonicalPath() : 파일의 정규경로를 리턴
- File getCanonicalFile() : 파일의 정규경로를 File객체로 리턴
