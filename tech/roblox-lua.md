# roblox - lua script

https://www.youtube.com/playlist?list=PL6Zm_z30gEKRhtuWuq6ZhJCJ2BsgkVgaL

## 이벤트
함수 인텔리센스에서 앞에 번개모양 아이콘 있는것들은 이벤트임

```lua
part.Touched:Wait()  -- 이벤트 발생할때까지 대기
part.Touched:Connect(함수)  -- 이벤트 발생시 호출

local touchedPart = part.Touched:Wait()  -- touch 된 파트 리턴

-- Touched.Connect 에 사용
fuction touched(toutchedPart)  -- touched 된 파트 가 파라매터로
end
```

닿은 파트를 모두 사라지게 만들기. 용암 연출?
```lua
part.Touched:Connect(function (touchedPart)
    touched:Destroy()  -- 해당 파트 제거
end)
```

## FindFirstChild
- 직접접근해서 사용하려는 없으면 에러남. 근데 해당 부모에 `FindFirstChild` 로 찾아서 없으면 nil 이 나옴
```lua
-- 파트에 터치된 캐릭터 찾기
local humanoid = touchedPart.Parent:FindFirstChild("Humanoid")  -- 모든 캐릭터는 Humanoid 라는 오브젝트가 있음 이걸로 캐릭터인지 판별
```

## FindFirst 시리즈
```lua
-- 이름으로 찾기
part:FindFirstChild("Humanoid", true)  -- true 를 하면 자식 노드까지 찾는다. 느림
part:FindFirstAncestor("Workspace")  -- 부모 찾기

-- 클래스로 찾기
game.ServerStorage:FindFirstChildOfClass("Part")  -- 객체 프로퍼티에 Data - class name 을 기준으로 찾음. 정확한 클래스명
game.ServerStorage:FindFirstChildWhichIsA("BasePart")  -- oop 의 부모 클래스로도 찾기 가능합
```
