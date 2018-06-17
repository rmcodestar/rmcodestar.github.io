---
layout: post
title: 소수판별법
category: 알고리즘
tag: [알고리즘, 소수]
---

## 소수 

위키에서의 소수 정의는 아래와 같다

>  소수(prime number)는 자신보다 작은 두 개의 자연수를 곱하여 만들 수 없는, 1보다 큰 자연수이다.
>  1과 그 수 자신 이외의 자연수로는 나눌 수 없는 자연수로 정의하기도 한다



예를 들면, 10 미만의 소수로는 2, 3, 5, 7이 있다.

소수의 패턴을 보면 2를 제외한 짝수인 자연수는 소수가 될 수 없다.


***

## 소수인지 판별하는 방법

특정 숫자 N이 소수인지 판별하기 위해서는 약수가 1과 N(자기자신)이 있는 경우인지를 판별하면 된다.

즉, 2 ~ (N-1)범위의 숫자 중 약수(divisor)가 있다면 소수가 아니다. (합성수라고도 한다)



간단하게 알고리즘을 작성하면 아래와 같다.

```Java
public boolean isPrime(int N) {
    if(N <= 2) {
        return (N % 2 == 0); //1은 소수가 아니고, 2은 소수인 경우에 대한 예외처리
    }

    for(int divisor = 2; divisor <= N - 1; divisor ++) {
        if(N % divisor == 0) {
            return false;	//2 ~ (N - 1) 중의 수로 나누어지므로 소수가 아님
        }
    }

    return true;   // 소인수가 없으므로 소수 판별
}
```

* 관련 알고리즘 문제 : [백준 - 소수 찾기](https://www.acmicpc.net/problem/1978)



## [Trial division](https://en.wikipedia.org/wiki/Trial_division) 

위 알고리즘에서는 (1, N-1) 범위에서 약수를 찾았으나 사실은 범위를 더 줄일 수 있다.

제일 작은 약수를 찾기만 하면 되기 때문에 (1, sqrt(N))으로 범위를 줄일 수 있다.

***

## 소수 찾는 방법

간단한 방법 중 하나인 [에라토스테네스의 체](https://ko.wikipedia.org/wiki/%EC%97%90%EB%9D%BC%ED%86%A0%EC%8A%A4%ED%85%8C%EB%84%A4%EC%8A%A4%EC%9D%98_%EC%B2%B4)를 이용하면 소수 목록을 구할 수 있다

탐색범위는 2부터 시작하며, 해당 숫자가 소수인지 판별하고 자신을 제외한 배수를 탐색후보에서 제거하는 식이다.



간단하게 소스를 작성하면

```Java
public void printPrimeNumber(int from, int to) {
        List<Boolean> inputs = new ArrayList<>();

        for(int index = 0; index< 2; index++) {
            inputs.add(false);
        }

        for (int index = 2; index <= to; index++) {
            inputs.add(true);
        }

        for (int i = 2; i <= Math.sqrt(to); index++) {
            if(isPrime(i) && inputs.get(i)) {
                for(int j = 2; j * i <= to; j++) { //해당 숫자의 배수를 탐색 후보에서 제외
                    inputs.set(j * i, false); 
                }
            }
        }

        for(int index = from; index<=to; index++) {
            if(inputs.get(index)) {
                System.out.println(index); // (from, to) 범위 내 소수 출력
            }
        }
}
```

* 관련 알고리즘 문제 : [백준 - 소수 구하기](https://www.acmicpc.net/problem/1929)



## reference

* [칸아카데미의 '소수 판별법' 강의](https://ko.khanacademy.org/computing/computer-science/cryptography/comp-number-theory/v/primality-test-challenge)
