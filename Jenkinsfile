node {


    stage concurrency: 1, name: 'Checkout'
    //setting up the configuration for the build
    setupConfiguration()

    //checking out the build
    sourceCheckout(checkoutURL)

    stage name : 'Build', concurrency : 1
    buildApplication()
    
    stage name : 'Copying the Artifacts'
    
    copingArtifactsBuildWise(compiledBinaryPath,buildArtifactsPath)
    
    stage name : 'Unit Test'
    
    //installNuGet(nugetExePath)
    //runUnitTest()
    runMSTestUnitTest()
    stage name : 'Static Code Analysis'
    
    runStaticCodeAnalysis()
}

def setupConfiguration()
{
    // GIT Checkout URL
    checkoutURL = 'https://github.com/deadlockV/SampleMVCApp.git'
    // Current Workspace path
    workSpaceDir = pwd()
    
     CREDENTIALS = 'infostretch_vimalID'
    
    config = null
    
    if (!(config?.trim())) {
                    config = "Release"
                }

    PackagesDir = workSpaceDir + '\\packages'
    OutDir = workSpaceDir + '\\bin\\Debug'
    // Path to NuGet exe downloaded into current workspace
    nugetTargetPath = workSpaceDir + "\\nuget"
    nugetExePath = nugetTargetPath + "\\nuget.exe"
    sourceNugetExe = 'http://nuget.org/nuget.exe'
    compiledBinaryPath = workSpaceDir + "\\SampleMVCApp\\bin"
    buildArtifactsPath = null
    if (!(buildArtifactsPath?.trim())) {
                    
                    buildArtifactsPath = workSpaceDir + "\\BuildArtifacts\\${env.BUILD_NUMBER}\\"
                }
}

def sourceCheckout(gitCheckoutURL)
{
    try
    {
        checkout changelog: false,
                poll: false,
                scm: [$class: 'GitSCM',
                      branches: [[name: '*/master']],
                      doGenerateSubmoduleConfigurations: false,
                      extensions: [[$class: 'SubmoduleOption',
                                    disableSubmodules: false,
                                    recursiveSubmodules: true,
                                    reference: '',
                                    trackingSubmodules: true],
                                   [$class: 'CloneOption',
                                    noTags: true, reference: '',
                                    shallow: true, timeout: 20]],
                      submoduleCfg: [],
                      userRemoteConfigs: [[credentialsId: CREDENTIALS,
                                           url: gitCheckoutURL]]]

       
        echo "Git Checkout Successful  "
    }catch (exception) {
        error "Checkout failed : ${exception}"
    }

    }


def runUnitTest()
{

   try
   {
       
       
       

   isNUnitRunnerExists = fileExists '${PackagesDir}\\NUnit.Runners.2.6.4\\tools\\nunit-console.exe'
    if(!isNUnitRunnerExists)  
    {
        echo "Installing NUnit Runner Version 2.6.2"
    //dir(nugetTargetPath){
   
        bat """REM Unit tests

        nuget.exe %nuget% install NUnit.Runners -OutputDirectory "${PackagesDir}"
        "${PackagesDir}\\NUnit.ConsoleRunner.3.2.0\\tools\\nunit3-console.exe" /config:${config} \"${workSpaceDir}\\SampleMVCApp.Tests\\bin\\${config}\\SampleMVCApp.Tests.dll\""""
        
        // Archieve the Unit Test result
    //    archieveFiles('console-test.xml')
    }
   }catch (exception) {
        error "Error in running Unit Test : ${exception}"
    }
    
//}





}

def runMSTestUnitTest()
{

   try
   {
     def mstesttool = tool name: 'mstest', type: 'org.jenkinsci.plugins.MsTestInstallation'
        echo "${mstesttool}"
        bat """@echo Off

            REM Build
            \"${mstesttool}\" /testcontainer:\"${workSpaceDir}\\SampleMVCApp.Tests\\bin\\${config}\\SampleMVCApp.Tests.dll\""""
    echo "MS Test execution successful"
    
   }catch (exception) {
        error "Error in running MS Unit Test : ${exception}"
    }
    
//}





}

def installNuGet(nugetExecutionPath)
{
    echo "Installing NuGet into : ${nugetExecutionPath}"
    try
    {
        isNuGetExists = fileExists 'nuget.exe'
        if(!isNuGetExists)
        {
            bat """	
            powershell -command Invoke-WebRequest ${sourceNugetExe} -OutFile nuget.exe"""
        }
    }
    catch (exception) {
        error "installing NuGet failed : ${exception}"
    }
}

   def buildApplication()
  {
      try
      {
          
        def msbuildtool = tool name: 'MSBuild', type: 'hudson.plugins.msbuild.MsBuildInstallation' 
        echo "${msbuildtool}"
        bat """@echo Off

            REM Build
            \"${msbuildtool}\" SampleMVCApp.sln  /t:Rebuild /p:Configuration=${config} /m /v:M /fl /flp:LogFile=msbuild.log;Verbosity=Normal /nr:false"""
        }
        catch (exception) {
        error "Build failed : ${exception}"
    }
  }
def archieveFiles(resultFile)
{
    archive resultFile
}
def copingArtifactsBuildWise(sourcePath,destPath)
{
    try
    {
        
    
        dir(destPath){
        echo "Copying files from ${sourcePath}  to  ${destPath}"
        bat "xcopy /s \"${sourcePath}\" \"${destPath}\""
        echo "Copying Artifacts Successful"
        }
    } 
    catch (exception) {
        error "Copying Artifacts failed : ${exception}"
        }
    
}

def runStaticCodeAnalysis()
{
    
    
    // FxCop Runner Jenkins Plug-in needs to be installed as a tool in Configure system through manage jenkins
    // If you need to analyze with custom ruleset, you can just type /ruleset:=YourCustomRuleSet.ruleset
    // FxCopCmd /f:SomeAssembly.dll /r:"C:\Rules Directory\SomeRules.dll" /o:OutputFile.xml
    
    try
    {
        def fxcop = tool name: 'FxCopCmd', type: 'org.jenkinsci.plugins.fxcop_runner.FxCopInstallation'
       
        bat """\"${fxcop}\\FxCopCmd.exe\" /gac /f:\"${buildArtifactsPath}\\SampleMVCApp.dll\" /o:\"${workSpaceDir}\\OutputFile.xml\""""
     
        echo "Static Code Analyzer completed successfully"
    }
      catch (exception) {
        error "Static Code Analyzer failed : ${exception}"
        }
} 