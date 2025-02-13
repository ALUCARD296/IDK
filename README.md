local Players = game:GetService("Players")
local Http = game:GetService("HttpService")
local TPS = game:GetService("TeleportService")
local LocalPlayer = Players.LocalPlayer
local Api = "https://games.roblox.com/v1/games/"

local function createUI()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Parent = game.CoreGui

    local MainFrame = Instance.new("Frame")
    MainFrame.Size = UDim2.new(0, 300, 0, 200)
    MainFrame.Position = UDim2.new(0.5, -150, 0.4, -90)
    MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    MainFrame.BorderSizePixel = 0
    MainFrame.Parent = ScreenGui

    local CloseButton = Instance.new("TextButton")
    CloseButton.Size = UDim2.new(0, 25, 0, 25)
    CloseButton.Position = UDim2.new(1, -30, 0, 5)
    CloseButton.BackgroundTransparency = 1
    CloseButton.Text = "-X"
    CloseButton.TextScaled = true
    CloseButton.Font = Enum.Font.GothamBold
    CloseButton.TextColor3 = Color3.fromRGB(255, 50, 50)
    CloseButton.Parent = MainFrame
    CloseButton.MouseButton1Click:Connect(function()
        ScreenGui:Destroy()
    end)

    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(1, -10, 0, 30)
    Title.Position = UDim2.new(0, 5, 0, 5)
    Title.BackgroundTransparency = 1
    Title.Text = "Choose Language | اختر لغتك"
    Title.TextScaled = true
    Title.Font = Enum.Font.GothamBold
    Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Title.Parent = MainFrame

    local ArabicButton = Instance.new("TextButton")
    ArabicButton.Size = UDim2.new(0.9, 0, 0, 40)
    ArabicButton.Position = UDim2.new(0.05, 0, 0.3, 0)
    ArabicButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    ArabicButton.Text = "العربية"
    ArabicButton.TextScaled = true
    ArabicButton.Font = Enum.Font.GothamBold
    ArabicButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    ArabicButton.Parent = MainFrame

    local EnglishButton = Instance.new("TextButton")
    EnglishButton.Size = UDim2.new(0.9, 0, 0, 40)
    EnglishButton.Position = UDim2.new(0.05, 0, 0.55, 0)
    EnglishButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    EnglishButton.Text = "English"
    EnglishButton.TextScaled = true
    EnglishButton.Font = Enum.Font.GothamBold
    EnglishButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    EnglishButton.Parent = MainFrame

    local LoadingLabel = Instance.new("TextLabel")
    LoadingLabel.Size = UDim2.new(1, 0, 0, 30)
    LoadingLabel.Position = UDim2.new(0, 0, 0.85, 0)
    LoadingLabel.BackgroundTransparency = 1
    LoadingLabel.Text = ""
    LoadingLabel.TextScaled = true
    LoadingLabel.Font = Enum.Font.GothamBold
    LoadingLabel.TextColor3 = Color3.fromRGB(255, 255, 50)
    LoadingLabel.Parent = MainFrame

    -- Add the loading bar frame
    local LoadingBarFrame = Instance.new("Frame")
    LoadingBarFrame.Size = UDim2.new(0.9, 0, 0, 5)
    LoadingBarFrame.Position = UDim2.new(0.05, 0, 0.95, 0)
    LoadingBarFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    LoadingBarFrame.Parent = MainFrame

    local LoadingBar = Instance.new("Frame")
    LoadingBar.Size = UDim2.new(0, 0, 1, 0)
    LoadingBar.BackgroundColor3 = Color3.fromRGB(0, 0, 255)
    LoadingBar.Parent = LoadingBarFrame

    -- Tween service for animation
    local TweenService = game:GetService("TweenService")
    local loadingTween = TweenService:Create(LoadingBar, TweenInfo.new(2, Enum.EasingStyle.Linear, Enum.EasingDirection.Out, -1, true), {Size = UDim2.new(1, 0, 1, 0)})

    local function showServerButtons(language)
        Title.Text = language == "Arabic" and "اختر السيرفر" or "Choose Server"
        ArabicButton.Visible = false
        EnglishButton.Visible = false

        local LowServerButton = Instance.new("TextButton")
        LowServerButton.Size = UDim2.new(0.9, 0, 0, 40)
        LowServerButton.Position = UDim2.new(0.05, 0, 0.3, 0)
        LowServerButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        LowServerButton.Text = language == "Arabic" and "الانتقال إلى سيرفر بعدد لاعبين قليل" or "Low Server"
        LowServerButton.TextScaled = true
        LowServerButton.Font = Enum.Font.GothamBold
        LowServerButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        LowServerButton.Parent = MainFrame

        local RandomServerButton = Instance.new("TextButton")
        RandomServerButton.Size = UDim2.new(0.9, 0, 0, 40)
        RandomServerButton.Position = UDim2.new(0.05, 0, 0.55, 0)
        RandomServerButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        RandomServerButton.Text = language == "Arabic" and "الانتقال إلى سيرفر عشوائي" or "Random Server"
        RandomServerButton.TextScaled = true
        RandomServerButton.Font = Enum.Font.GothamBold
        RandomServerButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        RandomServerButton.Parent = MainFrame

        local function getServers()
            local placeId = game.PlaceId
            local servers = Api .. placeId .. "/servers/Public?sortOrder=Asc&limit=100"
            local raw = game:HttpGet(servers)
            return Http:JSONDecode(raw).data
        end

        local function teleportToServer(serverId)
            LoadingLabel.Text = "Loading..."
            loadingTween:Play() -- Start loading animation
            wait(2) -- تأخير لتحسين التجربة
            TPS:TeleportToPlaceInstance(game.PlaceId, serverId, LocalPlayer)
        end

        LowServerButton.MouseButton1Click:Connect(function()
            local servers = getServers()
            if #servers > 0 then
                teleportToServer(servers[1].id)
            end
        end)

        RandomServerButton.MouseButton1Click:Connect(function()
            local servers = getServers()
            if #servers > 0 then
                local randomServer = servers[math.random(1, #servers)]
                teleportToServer(randomServer.id)
            end
        end)
    end

    ArabicButton.MouseButton1Click:Connect(function()
        showServerButtons("Arabic")
    end)

    EnglishButton.MouseButton1Click:Connect(function()
        showServerButtons("English")
    end)
end

createUI()
