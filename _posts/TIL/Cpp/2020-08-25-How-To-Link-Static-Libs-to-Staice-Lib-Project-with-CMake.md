---
layout: post
title: TIL - CMake사용하여 정적 라이브러리 프로젝트에 다른 정적 라이브러리들을 연결하기. 
date:   2020-08-25 16:06:46
description: 
img: TIL/Thumbnail.jpg # Add image post (optional)
tags: [CMake]
author: Junyoung Kim # Add name author (optional)
---

# 개요
최근 내 엔진을 깃허브에 올리기 위해, 메타 빌드 시스템을 도입하기로 결정했다. 처음에는 Lua를 그대로 사용한다는 간편함 때문에 Premake5를 사용하려 했으나, 사용자가 너무나 적고 아직 알파 단계라, 보편적으로 많이 사용되는 CMake를 배워보고 도입하기로 결정했다.

CMakeLists를 작성하면서 어떻게 생성되는지 테스트 하는 도중, .lib 파일들이 내 엔진(정적 라이브러리 프로젝트)의 Librarian > General 의 Additional Dependencies와 Additional Libraries Directories 칸에 제대로 연결이 안되는 문제들이 발생했다. 
원인을 알아보니, 원래 IDE단에서 정적 라이브러리들을 정적 라이브러리 프로젝트 연결하는 기능은 MS의 Visual Studio에서만 가능하단다. VS 말곤 지원하지 않는 기능이니, CMake도 당연스레 해당 기능을 제공하지 않는다고 한다. 

하지만 늘 그랬듯이 해결책은 존재한다.

```plaintext
######################################################
#   Syntax
######################################################

target_link_directories(<target name> 
    <Acess modifier: PUBLIC, PRIVATE>
			<directory 1>
			<directory 2>
			<directory 3>
			<directory 4>
)

target_link_libraries(<target name> 
    <Acess modifier: PUBLIC, PRIVATE>
        <lib name 1>
        <lib name 2>
        <lib name 3>
)

######################################################
# Example
######################################################

target_link_directories(${ProjectName} 
    PUBLIC
			${CMAKE_CURRENT_SOURCE_DIR}/SDL2/lib/
      ${CMAKE_CURRENT_SOURCE_DIR}/SDL2_image/lib/
      ${CMAKE_CURRENT_SOURCE_DIR}/SDL2_mixer/lib/
)

target_link_libraries(${ProjectName} 
    PUBLIC
				SDL2
        SDL2_image
        SDL2_mixer
)

```
위 두 함수를 사용하면, 서드파티 정적 라이브러리들을 자신이 생성한 정적 라이브러리 프로젝트에 연결할 수 있다. 물론 비주얼 스튜디오에선 표시가 제대로 뜨지 않고, CMake를 사용할때 올바른 라이브러리를 연결하는 방식은 아니지만, 작동만 하면 됐다 ㅋㅋㅋ

`target_link_directories` 함수와 `target_link_libraries` 함수는 비슷한 신택스를 공유한다.<br>
공통적으로 타겟 이름과 2. 접근 제한자가 필요하다.

다른 점은, 아래와 같다.<br>
 `target_link_directories` : 사용하고자 하는 라이브러리에 대한 절대 or 상대 경로가 필요.<br>
 `target_link_libraries`   : 사용하고자 하는 라이브러리의 이름 필요.
 </t> 이름을 " "로 감쌀 필요는 없음.

<br>
다른 환경을 사용할 경우에는, ' AR ' 같은 라이브러리안 툴을 활용해보자.<br><br>
윈도우즈 환경이라면 그냥 비주얼 스튜디오를 쓰자.<br>Visual Studio가 짱짱이다. MS 만세! 

