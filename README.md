import java.util.zip.ZipFile
import java.nio.file.Paths
import java.nio.file.Files

// Define a custom task to prepare the environment
task prepareEnvironment {
    // Define the task configuration
    doLast {
        // Define variables for the download URL, zip file name, target directory, etc.
        def zipVersion = project.hasProperty('zipVersion') ? project.zipVersion : '1.0.0'
        def zipFileName = "your-zip-file-$zipVersion.zip"
        def targetDir = file("$buildDir/unzipped")

        // Check if the zip file is already downloaded
        if (!file("$buildDir/$zipFileName").exists()) {
            // If not downloaded, proceed with download, unzip, and copy
            def zipUrl = "https://your.nexus.repo.url/repository/your-repo/$zipFileName"

            // Create the target directory if it doesn't exist
            targetDir.mkdirs()

            // Download the zip file from Nexus
            ant.get(src: zipUrl, dest: file("$buildDir/$zipFileName"))

            // Extract the downloaded zip file
            project.copy {
                from(zipTree("$buildDir/$zipFileName"))
                into(targetDir)
            }

            // Copy files to a particular folder
            copy {
                from targetDir
                into 'your/target/folder'
                // Define any necessary file or directory filtering
            }
        } else {
            // If already downloaded, skip the download operation
            println "Skipping download, project is already downloaded."
        }
    }
}

// Configure dependencies between tasks
prepareEnvironment.mustRunBefore configurations.springPlatform.getTaskDependencyFromProjectDependency(false, 'compileJava')
prepareEnvironment.mustRunBefore configurations.springBootRuntime.getTaskDependencyFromProjectDependency(false, 'compileJava')
