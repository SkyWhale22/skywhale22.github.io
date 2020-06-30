---
layout: post
title: 🎮게임 엔진 개발기(3) - MVC 패턴을 이용한 게임 엔진 아키텍처 파트 2
date: 2020-06-28 03:18:00
description: 
img: GameEngine/Post2/BackGround.jpg # Add image post (optional)
tags: [game engine, c++]
author: Junyoung Kim # Add name author (optional)
---
# **서론**
지난 시간에는 ApplicationLayer 클래스와 System 인터페이스 클래스, 그리고 플랫폼별 클래스들의 구현과 팩토리 메소드를 사용한 생성 방법에 관해서 이야기를 하였다. 이번 포스트에선 GameLayer에 관해서 자세히 설명하고자 한다.
파트 1을 아직 읽지 않은 사람들은 우선, 지난 포스트를 읽고 오길 바란다.
<br>
<br>

# **게임 상태 관리하기**


- 게임 레이어가 하는 중요한 일: 게임의 상태를 계속 확인하는 작업.
- Game State: 게임마다 GameState에서 다루는 항목이 다를 수 있어, 완벽한 솔루션은 없음. 게임이 제대로 작동하는 것이 가장 중요하니 적당한 방식을 사용하면 된다. 성능이 중요한 사항이 아니라면, 보통 배열, 해쉬 맵 같은 기존에 널리 알려진 자료 구조를 사용하는 방법으로도 충분하다.
- 컴퓨팅에 있어 주요한 트레이드-오프는 속도와 메모리 사용량. 
  메모리가 넉넉하다면, CPU 사용률을 줄이기 위해서 여러 자료구조중 고려해보는것도 좋음.
  모바일 게이밍 환경에선, CPU 사용률을 줄이면 배터리 사용량 또한 줄일 수 있음.
- 게임이 느려지기 시작할때 프로파일링 후, 게임이 느려지게 하는 원인을 최적화을 하면 된다. 예를 들면, CPU 사용률이 너무 높으면, 메모리 사용률을 높이면 되고, 반대로 메모리 사용률이 너무 높으면 CPU 사용률을 높이면 된다. 처음 부터 모든 항목을 최적화 하려고 하지 말고, 항당 '적당히'를 유지하자. 너무 이른 최적화는 모든 악의 뿌리(Premature Optimization Is the Root of All Evil)임을 꼭 기억하자.


- 단 한개의 Game State만이 게임의 제어에 대한 권한을 가지고 있어야한다. 이는 여러 플레이이어간 월드 동기화를 유지하는데 필요하다. 


# **게임 레이어가 하는 다른 일들**
- 게임 레이어가 하는 다른 일들: 게임 물리, 게임 프로세스, 게임 이벤트들의 관리 

- 게임 물리: 게임 월드내에 있는 오브젝트들이 서로 어떻게 상호작용을 하는지. 현실의 물리 법칙과 최대한 흡사하게   
           만드는것도 좋지만, 게임의 재미를 위해서 여러 규칙들을 바꾸는 것도 좋음. 현실적인 물리를 구현하면, 
           유저가 어떤 상호작용이 일어날지 쉽게 예상할 수 있어, 여러 선택지를 취할 수 있다. 하지만, 높은 
           곳에서 떨어져 다리 뼈가 뿌러지고 몇주간 병원에 입원해야하는 등의 사실성까지 포함하면, 좋은 게임 
           플레이 경험은 아닐것이다.

- 게임 이벤트: 게임 내에서 발생할 수 있는 여러 이벤트들. 예를 들어, 플레이어 캐릭터의 체력이 적을때, 낮은 체력
             임을 나타내는 애니메이션이 실행되거나, 적 AI들이 행동하는 패턴이 달라지거나 하는 등의 상호작용.
             Game State를 이벤트 시스템과 연결 해놓는것은 좋은 생각.
             Game State는 다른 시스템들의 저장소와 마찬가지임으로, 다른 여러 시스템들이 서로 직접적으로
             커플링 되지 않도록 유지하는것이 중요.

- 게임 프로세스: 게임 프로세스는 게임 내에서 여러 프레임에 걸쳐 일어나는 모든 행동이나 절차.
               예) 캐릭터가 특정 좌표까지 이동하는 행동이나, AI들의 길찾기 행동을 취하는 것들.
               여러 게임 프로세스들을 엮어 새로운 게임 프로세스를 만드는 등, 굉장히 강력한 기능이 될 것.
               게임 이벤트가 이런 프로세스들을 만드는데 사용될 수 있음.

# **GameLayer 인터페이스 선언**
```cpp
/*
*  GameLayer.h
* 
*  Copyright (c) 2020 Junyoung Kim. All rights reserved.
*/

class IGameLayer
{
public:
    virtual ~IGameLayer() = 0 {}
    virtual std::string_view GetGameName() const = 0;
};
```

이렇게 한 뒤, `ApplicationLayer`에 게임 레이어를 생성하는 팩토리 메소드를 넣어두면 됨.
```cpp
/*
*  ApplicationLayer.h
*
*  Copyright (c) 2020 Junyoung Kim. All rights reserved.    
*/

class ApplicationLayer
{
private:
    std::unique_ptr<IGameLayer> m_pGameLayer;

public:
    virtual std::unique_ptr<IGameLayer> CreateGameLayer() = 0;
};
```

생성 부분을 구현한뒤, `Initialize()` 함수에서 생성후, 초기화 하는 과정을 거치면 됨. 종료시 `Shutdown()` 함수에서도 처리를 해줘야하는것을 잊지말자.

그 뒤, 게임 프로젝트 파트에서 `ApplicationLayer`와 `IGameLayer`를 상속받는 클래스를 만들어주면된다.
```cpp
/*
* Game.h
*
* Copyright (c) 2020 Junyoung Kim. All rights reserved.
*/
#pragma once
#include <ApplicationLayer.h>

class GameLogic : public IGameLayer
{
public:
    virtual ~GameLogic() override {}
    virtual std::string_view GetGameName() const override
    {
        return "Game";
    }
};

class GameApp : public ApplicationLayer
{
public:
    virtual ~GameApp() override {}
    virtual std::unique_ptr<IGameLayer> CreateGameLayer() override
    {
        return std::make_unique<GameApp>();
    } 
}
```