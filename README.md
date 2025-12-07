local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

_G.HeadSize = 20
_G.HighlightEnabled = false
_G.HeadColor = Color3.fromRGB(0, 0, 255)
_G.ToggleKey = Enum.KeyCode.H

local OriginalProperties = {}

local ScreenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
ScreenGui.Name = "碰撞箱GUI"

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 320, 0, 300)
Frame.Position = UDim2.new(0.5, -160, 0.3, -150)
Frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Frame.BorderSizePixel = 0
Frame.Active = true
Frame.Draggable = true
Frame.Parent = ScreenGui

local function CreateLabel(text, posY)
    local label = Instance.new("TextLabel", Frame)
    label.Size = UDim2.new(1, -20, 0, 30)
    label.Position = UDim2.new(0, 10, 0, posY)
    label.Text = text
    label.TextColor3 = Color3.fromRGB(255,255,255)
    label.BackgroundTransparency = 1
    label.Font = Enum.Font.Gotham
    label.TextSize = 18
    return label
end

local function CreateButton(labelVar, posY, callback)
    local btn = Instance.new("TextButton", Frame)
    btn.Size = UDim2.new(0.8,0,0,30)
    btn.Position = UDim2.new(0.1,0,0,posY)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 16
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.BackgroundColor3 = Color3.fromRGB(70,70,70)

    local function UpdateText()
        if labelVar == "Highlight" then
            btn.Text = "碰撞箱: "..(_G.HighlightEnabled and "開啟" or "關閉").." (快捷鍵: "..tostring(_G.ToggleKey.Name)..")"
        elseif labelVar == "SetKey" then
            btn.Text = "設定快捷鍵"
        elseif labelVar == "Close" then
            btn.Text = "關閉腳本"
        end
    end

    btn.MouseButton1Click:Connect(function()
        callback()
        UpdateText()
    end)

    UpdateText()
    return btn
end

local function CreateSlider(labelText, posY, min, max, default, callback)
    local label = CreateLabel(labelText.." : "..default, posY)
    local slider = Instance.new("TextBox", Frame)
    slider.Size = UDim2.new(0.8,0,0,25)
    slider.Position = UDim2.new(0.1,0,0,posY+30)
    slider.Text = tostring(default)
    slider.TextColor3 = Color3.fromRGB(0,0,0)
    slider.ClearTextOnFocus = false
    slider.FocusLost:Connect(function()
        local value = tonumber(slider.Text)
        if value then
            callback(value)
            label.Text = labelText.." : "..value
        end
    end)
end

local HighlightButton = CreateButton("Highlight", 10, function()
    _G.HighlightEnabled = not _G.HighlightEnabled
    UpdateHighlights()
end)

CreateSlider("大小", 70, 1, 100, _G.HeadSize, function(v) _G.HeadSize = v end)

local KeyPromptLabel
CreateButton("SetKey", 120, function()
    if not KeyPromptLabel then
        KeyPromptLabel = CreateLabel("請按下新的快捷鍵...", 150)
    else
        KeyPromptLabel.Text = "請按下新的快捷鍵..."
    end

    local conn
    conn = UserInputService.InputBegan:Connect(function(input, gp)
        if gp then return end
        if input.UserInputType == Enum.UserInputType.Keyboard then
            _G.ToggleKey = input.KeyCode
            KeyPromptLabel.Text = "設置快捷鍵: "..tostring(_G.ToggleKey.Name)
            if HighlightButton then
                HighlightButton.Text = "碰撞箱: "..(_G.HighlightEnabled and "開啟" or "關閉").." (快捷鍵: "..tostring(_G.ToggleKey.Name)..")"
            end
            conn:Disconnect()
        end
    end)
end)

CreateButton("Close", 200, function()
    _G.HighlightEnabled = false
    UpdateHighlights()
    if ScreenGui then ScreenGui:Destroy() end
end)

function UpdateHighlights()
    for _,v in next, Players:GetPlayers() do
        if v ~= LocalPlayer then
            local hrp = v.Character and v.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                if not OriginalProperties[v] then
                    OriginalProperties[v] = {
                        Size = hrp.Size,
                        Transparency = hrp.Transparency,
                        Material = hrp.Material,
                        CanCollide = hrp.CanCollide,
                        BrickColor = hrp.BrickColor
                    }
                end
                if _G.HighlightEnabled then
                    hrp.Size = Vector3.new(_G.HeadSize,_G.HeadSize,_G.HeadSize)
                    hrp.Transparency = 0.9
                    hrp.Material = Enum.Material.Neon
                    hrp.BrickColor = BrickColor.new(_G.HeadColor)
                    hrp.CanCollide = false
                else
                    local orig = OriginalProperties[v]
                    hrp.Size = orig.Size
                    hrp.Transparency = orig.Transparency
                    hrp.Material = orig.Material
                    hrp.CanCollide = orig.CanCollide
                    hrp.BrickColor = orig.BrickColor
                end
            end
        end
    end
end

UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == _G.ToggleKey then
        _G.HighlightEnabled = not _G.HighlightEnabled
        if HighlightButton then
            HighlightButton.Text = "碰撞箱: "..(_G.HighlightEnabled and "開啟" or "關閉").." (快捷鍵: "..tostring(_G.ToggleKey.Name)..")"
        end
        UpdateHighlights()
    end
end)

RunService.RenderStepped:Connect(function()
    for _,v in next, Players:GetPlayers() do
        if v ~= LocalPlayer then
            local hrp = v.Character and v.Character:FindFirstChild("HumanoidRootPart")
            if hrp and not OriginalProperties[v] then
                OriginalProperties[v] = {
                    Size = hrp.Size,
                    Transparency = hrp.Transparency,
                    Material = hrp.Material,
                    CanCollide = hrp.CanCollide,
                    BrickColor = hrp.BrickColor
                }
            end
        end
    end
end)
