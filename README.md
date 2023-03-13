# GitHub Copilot in GitHub Codespaces

GitHub Codespaces 안에서 GitHub Copilot과 Bing AI 데모용으로 사용했던 리포지토리입니다.


## 관련 발표 영상

* [DDD(Daegu Developer Day)](https://www.youtube.com/live/HIQNLITA6dE?feature=share&t=6212)
* Azure Open Source Day 2023 Seoul


## 시작하기

> **DISCLAIMER**:
> 
> 이 리포지토리는 Generative AI(생성 ID)의 여러 종류 중 하나인 [GitHub 코파일럿][gh copilot]과 [ChatGPT][chatgpt]에서 사용한 프롬프트만을 모아뒀습니다. 프롬프트에 따라 코파일럿과 ChatGPT가 생성해 주는 결과는 상황에 따라 다를 수 있으므로, 이 프롬프트대로 따라한다고 하더라도 실제 발표 영상과 다른 결과가 나올 수 있습니다.


### 코드스페이스 설정

1. 코드스페이스 운영체제 확인

    ```bash
    cat /etc/os-release
    ```

2. 코드스페이스 환경 변수 확인

    ```bash
    printenv -0 | sort -z | tr '\0' '\n'
    ```

3. 코드스페이스 업데이트

    ```bash
    sudo apt update \
        && sudo apt upgrade -y \
        && sudo apt clean -y \
        && sudo rm -rf /var/lib/apt/lists/*
    ```

4. 애저 펑션 코어 툴 설치

    ```bash
    npm install -g azure-functions-core-tools@4 --unsafe-perm true
    ```

5. 파워셸 설치

    ```bash
    dotnet tool install --global PowerShell
    ```

### 애저 펑션 프로젝트 생성

1. .NET 솔루션 생성

    ```bash
    dotnet new sln -n FuncApp
    ```

2. 애저 펑션 프로젝트 생성

    ```bash
    func init ApiApp
    ```

3. 애저 펑션과 솔루션 연결

    ```bash
    dotnet sln add ApiApp
    ```

4. 애저 펑션 앱 빌드

    ```bash
    dotnet restore && dotnet build
    ```

   만약 빌드 에러 생기면 `ApiApp.csproj` 파일을 아래와 같이 수정 후 다시 빌드

    ```xml
    <ItemGroup>
      <!-- ⬇️⬇️⬇️ 아래 버전 값을 "*"로 변경 ⬇️⬇️⬇️ -->
      <PackageReference Include="Microsoft.NET.Sdk.Functions" Version="4.1.1" />
      <!-- ⬆️⬆️⬆️ 아래 버전 값을 "*"로 변경 ⬆️⬆️⬆️ -->
    </ItemGroup>
    ```

5. 애저 펑션앱 디렉토리로 이동

    ```bash
    pushd ApiApp
    ```

6. 애저 펑션 HTTP 엔드포인트 추가

    ```bash
    func new -n GreetingHttpTrigger
    ```

7. 애저 평선 실행

    ```bash
    func start
    ```

8. 리포지토리 루트 디렉토리로 이동

    ```bash
    popd
    ```

### OpenAPI 확장 기능 추가

1. 코드스페이스에서 작동 가능하게끔 환경 설정 수정

    ```bash
    pwsh -c "Invoke-RestMethod https://aka.ms/azfunc-openapi/add-codespaces.ps1 | Invoke-Expression"
    ```

2. `ApiApp.csproj` 파일에 GitHub 코파일럿으로 NuGet 패키지 추가

    ```xml
    <ItemGroup>
      <PackageReference Include="Microsoft.NET.Sdk.Functions" Version="*" />

      <!-- ⬇️⬇️⬇️ 아래 주석을 각각 한 줄씩 추가한 후 코파일럿이 NuGet 패키지 추천해주는 것 추가 ⬇️⬇️⬇️ -->
      <!-- install azure functions openapi extension for webjob -->
      <!-- install azure functions dependency injection -->
    </ItemGroup>
    ```

3. 애저 펑션앱 빌드

    ```bash
    dotnet restore && dotnet build
    ```

4. `GreetingHttpTrigger.cs` 파일에 GitHub 코파일럿으로 OpenAPI 데코레이터, 네임스페이스 및 비즈니스 로직 추가

    ```csharp
    ...
    using Newtonsoft.Json;
    // ⬇️⬇️⬇️ 아래 주석을 추가한 후 코파일럿이 추천해주는 라인 추가 ⬇️⬇️⬇️
    // add namespaces for openapi extension

    namespace ApiApp
    {
        ...
        [FunctionName("GreetingHttpTrigger")]
        // ⬇️⬇️⬇️ 아래 주석을 추가한 후 코파일럿이 추천해주는 라인 추가 ⬇️⬇️⬇️
        // add openapi decorators for operation, parameter and responses of OK, BadRequest and InternalServerError
        ...

            string name = req.Query["name"];
            // ⬇️⬇️⬇️ 아래 주석을 추가한 후 코파일럿이 추천해주는 라인 추가 ⬇️⬇️⬇️
            // if name is empty, return badrequestresult
            ...

            // ⬇️⬇️⬇️ 아래 주석을 추가한 후 코파일럿이 추천해주는 라인 추가 ⬇️⬇️⬇️
            // use try catch block to return internal server error result if exception is thrown
    ```

5. ChatGPT에 아래 프롬프트를 입력해서 애저 펑션의 IoC 컨테이너용 `Startup.cs` 파일 코드 생성

    ```text
    Azure Functions 앱에 사용할 IoC 컨테이너 용도로 Startup.cs 파일을 생성해 주세요. 이 때 네임스페이스는 ApiApp 입니다.
    ```

6. Startup.cs 파일에 GitHub 코파일럿으로 OpenAPI 네임스페이스 및 비즈니스 로직 추가

    ```csharp
    using Microsoft.Azure.Functions.Extensions.DependencyInjection;
    using Microsoft.Extensions.DependencyInjection;
    ...
    // ⬇️⬇️⬇️ 아래 주석을 각각 추가한 후 코파일럿이 추천해주는 라인 추가 ⬇️⬇️⬇️
    // add azure functions openapi extensions namespace for IOpenApiConfigurationOptions
    // add azure functions openapi extensions namespace for DefaultOpenApiConfigurationOptions
    
    [assembly: FunctionsStartup(typeof(ApiApp.Startup))]
    namespace ApiApp
    {
        public class Startup : FunctionsStartup
        {
            public override void Configure(IFunctionsHostBuilder builder)
            {
                ...
    
                // ⬇️⬇️⬇️ 아래 주석을 추가한 후 코파일럿이 추천해주는 라인 추가 ⬇️⬇️⬇️
                // Add a singleton instance of IOpenApiConfigurationOptions that is initiated from DefaultOpenApiConfigurationOptions.
    
                // ⬇️⬇️⬇️ 아래 주석을 추가한 후 코파일럿이 추천해주는 라인 추가 ⬇️⬇️⬇️
                // Add a property of includerequestinghostname set to false
            }
        }
    }
    ```

7. 애저 펑션앱 디렉토리로 이동

    ```bash
    pushd ApiApp
    ```

8. 애저 평선 실행

    ```bash
    func start
    ```

9. 리포지토리 루트 디렉토리로 이동

    ```bash
    popd
    ```

[gh copilot]: https://github.com/features/copilot
[chatgpt]: https://chat.openai.com
