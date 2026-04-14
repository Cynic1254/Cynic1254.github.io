---
title: "Katharsi — DevOps & tools"
order: 2
summary: >
  CI/CD pipeline and Steam deployment infrastructure for a 15-person BUAS
  capstone team. Built Jenkins automation, daily testing, and cross-platform
  releases that shipped two commercial Steam builds.
status: "Live on Steam"
tags:
  - Jenkins
  - Unreal Engine
  - Steam API
  - Groovy
  - Build automation
image: "/assets/images/Katharsi_Cover.png"
image_alt: "Katharsi game cover art"
stats:
  - value: "15"
    label: "Team members"
  - value: "2"
    label: "Steam releases"
  - value: "365+"
    label: "Days of automated tests"
link_detail_label: "Play on Steam"
link_detail_url: "https://store.steampowered.com/app/3365850/Katharsi/"
link_source_label: "View pipeline plugin on GitHub"
link_source_url: "https://github.com/Cynic1254/JenkinsDiscordTools"
---

## Overview

As part of my All year project at BUAS, I worked on Katharsi in a team of 15 people, 6 Designers, 6 Artists and 3 Programmers.

Katharsi is a puzzle adventure set in a corrupted Byzantine cathedral. With the help of your companion, cleanse and illuminate the path in front of you by harnessing the power of the light. Solve the puzzles that block your path and reveal the truth behind this story.

Besides general programming tasks due to the limited amount of programmers in the team, my main focus for this project was CI/CD. I set up and maintained the Jenkins pipeline that allowed my team to automatically test and deploy the project to steam, Featuring clear and helpful information in case of failed tests and other errors, sent directly to discord. As well as a clear overview of the history of tests and their success rates.

During production I also built a Steam Input integration plugin for the game, which later grew into a standalone plugin. See [Steam Input API plugin]({% link _projects/steam-input-plugin.md %}) for the full details.

## The Pipeline

One of the main concerns when running CI/CD is the limited amount of shared resources available among BUAS. The Jenkins servers at BUAS have a total of 3 slots available to run pipeline scripts, being shared across 4 years pipelines need to prioritize speed and stability as a main concern. If a pipeline get's stuck or takes a while to complete, that slot becomes unavailable for the duration.

To address this concern I split up the pipeline into 2 separate pipelines, one for daily testing and one for building and uploading the project. The testing pipeline is kept light to run, only performing tests using Unreal Engine's testing framework. The results of the test are then parsed and send to discord as a system report. Additionally the tests are also compiled as a junit compatible test file allowing for a report over time about the status of the tests.
![Failed Test results](/assets/images/Failed_Tests.png) ![Successful tests](/assets/images/Successful_Tests.png) ![Test Graph](/assets/images/Test_Graph.png)
```groovy
pipeline {
  stages {
    stage ("P4 Setup") { // Download the repository from the remote source
      steps {
        script {
          def newestChangeList = p4v.initGetLatestCL(env.P4USER, env.P4HOST)
          echo("NEWEST CHANGELIST: ${newestChangeList}")
          echo("${STAGE_NAME}")
          p4v.init(env.P4USER, env.P4HOST, env.P4WORKSPACE, env.P4MAPPING, newestChangeList, !true)
          echo("P4 synced to: ${env.P4_CHANGELIST}")
        }
      }
    }
    stage ("Testing") { // Run the automated testing command
      steps {
        script {
          echo("${STAGE_NAME}")
          
          JenkinsTools.Unreal.Init("${env.CONFIG}", "${env.PLATFORM}", "${env.ENGINEROOT}", "${env.PROJECT}")
          def testResult = JenkinsTools.Unreal.RunAutomationCommand("RunTests Project")
          JenkinsTools.DiscordMessage.FromTestResults(testResult).Send(env.DiscordWebhook)

          testResult.WriteXMLToFile("${env.WORKSPACE}/Logs/UnitTestsReport/Report.xml")

          junit allowEmptyResults: true, skipMarkingBuildUnstable: true, skipPublishingChecks: true, stdioRetention: '', testResults: 'Logs/UnitTestsReport/Report.xml'
        }
      }
    }
  }
  post {
    success {
      script {
        log("Build Succeeded!")
        JenkinsTools.DiscordMessage.Succeeded().Send(env.DiscordWebhook)
      }
    }
    unstable {
      script {
        log("Build Unstable!")
        JenkinsTools.DiscordMessage.Unstable().Send(env.DiscordWebhook)
      }
    }
    failure {
      script {
        log("Build Failed!")
        JenkinsTools.DiscordMessage.Failed().Send(env.DiscordWebhook)
      }
    }
    aborted {
      script {
        log("After Abort actions")
        JenkinsTools.DiscordMessage.Aborted().Send(env.DiscordWebhook)
        if(env.CLEANWORKSPACE.toBoolean()) {
          cleanWs()
        }
      }
    }
  }
}
```

`JenkinsTools` uses a custom Jenkins plugin that has been made by me, it's designed to simplify much of the logic for parsing test results and formatting discord webhook messages. It provides some sensible templates for discord messages that contain formatting for various notifications to be send. The code mostly provides simple formatting but for those curious the source can be found [here](https://github.com/Cynic1254/JenkinsDiscordTools)

The deployment pipeline alongside running the tests also builds and deploys the game to any external store, falling back to a google drive upload in the case of failing tests for download and debugging.
![Failed Build Result](/assets/images/Failed_Build.png) ![Successful Build Result](/assets/images/Sucessful_Build.png) ![Unstable Build Result](/assets/images/Unstable_Build.png)
```groovy
pipeline {
  stages {
    stage('P4-Setup') { // download the build from external sources
      steps {
        script {
          def newestChangeList = p4v.initGetLatestCL(env.P4USER, env.P4HOST)
          log("NEWEST CHANGELIST: ${newestChangeList}")
          log.currStage()
          p4v.init(env.P4USER, env.P4HOST, env.P4WORKSPACE, env.P4MAPPING, newestChangeList, !env.CLEANWORKSPACE)
          log("P4 synced to: ${env.P4_CHANGELIST}")
        }
      }
    }
    stage("UE5-Build") { // Build the game
      steps {
        script {
          log.currStage()
          win.makeWritable(env.PROJECTDIR)
          if(env.PLATFORM == "PS5") {
					      win.movePathFiles("\"${PROJECTDIR}Content\\Images\"", "\"${ENGINEROOT}Engine\\Platforms\\PS5\\Build\\sce_sys\"")
					      }
          ue5.buildPrecompiledProject(env.ENGINEROOT, env.PROJECTNAME, env.PROJECT, env.CONFIG, env.PLATFORM, env.OUTPUTDIR)
        }
      }
    }
    stage("Automation-Tests") { // run automated tests
      when { 
        expression { env.RUNTESTS.toBoolean() }
      }
      steps {
        script {
          log.currStage()
          ue5.runAutomationCommand("RunTests Project.")          // Tests to run
        }
      }
    }
    stage("Primary-DDS-Deployment") { // deploy to DDS
      steps {
        script {
          log.currStage()
          def platformDir = env.PLATFORM
          if(env.PLATFORM == "Win64") {
            platformDir = "Windows"
          }
          if(env.DDS == "Steam") {
            if(currentBuild.result == "UNSTABLE") {
              log.warning("====!!==== UNSTABLE BUILD - PUSH TO STEAM ABORTED, PUSHING TO GDRIVE REGARDLESS OF CONFIG TO ALLOW DEBUG ====!!====")
            }
            else if(env.PLATFORM == "PS5") {
              log.warning("====!!==== PS5 BUILD - NOT ALLOWED TO UPLOAD TO STEAM ====!!====")
            }
            else
            {
              discord.sendMessage(discord.createMessage(
              "Started push to steam",
              "white",
              [[name:"Push to Steam has been initiated, SteamGuard Authorization might be required", 
              value:"${env.BUILD_URL}"],
              [name:"Authorizer:", 
              value:"<------>"]],
              [text:"${env.JOB_BASE_NAME}"])
              , env.WebHook_BUILD)
              
              steam.init(env.STEAMUSR, env.STEAMCMD)
              
              def appManifest
              if (env.PLATFORM == "Linux"){
                appManifest = steam.createAppManifest("----", "----", "", "${env.JOB_BASE_NAME}", false, "", "${env.STEAMBRANCH}", env.OUTPUTDIR)
                steam.createDepotManifest("----", "${env.OUTPUTDIR}\\${platformDir}")
              } else {
                appManifest = steam.createAppManifest("----", "----", "", "${env.JOB_BASE_NAME}", false, "", "${env.STEAMBRANCH}", env.OUTPUTDIR)
                steam.createDepotManifest("----", "${env.OUTPUTDIR}\\${platformDir}")
              }
              
              try {
                steam.tryDeploy("${env.WORKSPACE}\\${appManifest}")
              }
              catch (Exception e) {
                dds_auth_timeout = true
                
                discord.sendMessage(discord.createMessage(
                "Steam Authorization Timed Out",
                "white",
                [[name:"Steam Authorization was not provided in time, Build has not been uploaded to steam, will be uploaded to GDrive instead", 
                value:"${env.BUILD_URL}"]],
                [text:"${env.JOB_BASE_NAME}"])
                , env.WebHook_BUILD)
                
                catchError(stageResult: 'UNSTABLE', buildResult: currentBuild.result) {
                  error("Mark Stage as Skipped")
                }
              }
            }
          }
          if(env.DDS == "Itch.io") {
            if(currentBuild.result == "UNSTABLE") {
            log.warning("====!!==== UNSTABLE BUILD - PUSH TO ITCH ABORTED, PUSHING TO GDRIVE REGARDLESS OF CONFIG TO ALLOW DEBUG ====!!====")
            }
            else if(env.PLATFORM == "PS5") {
              log.warning("====!!==== PS5 BUILD - NOT ALLOWED TO UPLOAD TO ITCH ====!!====")
            }
            else {
              itch.upload("${env.BUTLER}", env.BUTLER_API_KEY, "\"${env.OUTPUTDIR}\\${platformDir}\"", "${env.BUTLERTARGET}")
            }
          }
        }
      }
    }
    stage("GDrive-Deployment") { // Fall back to google drive upload if any of the tests fail
      steps {
        script {
          log.currStage()
				  if(env.PLATFORM == "Win64") {
				    zip.pack("${env.OUTPUTDIR}\\Windows", "${env.JOB_BASE_NAME}_${env.BUILD_NUMBER}", use7z = false)
				  }
				  else {
				    zip.pack("${env.OUTPUTDIR}\\${env.PLATFORM}", "${env.JOB_BASE_NAME}_${env.BUILD_NUMBER}", use7z = false)
				  }
          withCredentials([file(credentialsId: '----', variable: 'SECRETFILE')]) {
            python.runScript("${env.PYTHONGDRIVEUPLOAD}", "${SECRETFILE}\" \"${env.WORKSPACE}\\${env.JOB_BASE_NAME}_${env.BUILD_NUMBER}.zip\" \"${env.JOB_BASE_NAME}_${env.BUILD_NUMBER}.zip\" \"${env.GOOGLEDRIVEID}\" ${env.UPLOADSIZEMULTIPLIER}")
          }
        }
      }
    }
  }
  post {
    success {
      script {
        log("Build Succeeded!")
        discord.succeeded(env.CONFIG, "(${env.PLATFORM})", env.WEBHOOK_BUILD)
      }
    }
    unstable {
      script {
        log("Build Unstable!")
        discord.unstable(env.CONFIG, "(${env.PLATFORM})", env.WEBHOOK_BUILD)
      }
    }
    failure {
      script {
        log("Build Failed!")
        discord.failed(env.CONFIG, "(${env.PLATFORM})", env.WEBHOOK_BUILD)
      }
    }
    aborted {
      script {
        log("After Abort actions")
        if(env.CLEANWORKSPACE.toBoolean()) {
          cleanWs()
        }
      }
    }
  }
}
```
The pipeline supports building for various platforms including Windows, Linux and even PlayStation (given that the installed engine version supports this). It also allows uploading to most popular storefronts such as Steam and Itch. When uploading to steam a discord notification will be send with a notification that the uploader should enter a steam-guard code. The discord also receives notifications about whether or not the upload has succeeded or not.

## BUAS Jenkins admin

Starting this project I also took over the role of the Jenkins Admin for BUAS, this meant that besides maintaining my own team's pipeline I also maintained the jenkins server in general, making sure plugins stay up to date, help out other teams with any issues and ensure no pipelines run for an extended period of time. I also kept the systems themselves up to date and installed packages when requested (for example dev-kits for various platforms).

The biggest task I had to do for this was to setup a system that automatically refreshed the SSL certificates, this is something that the server lacked when I took over and when the certificates expired I took the opportunity to automate the process.