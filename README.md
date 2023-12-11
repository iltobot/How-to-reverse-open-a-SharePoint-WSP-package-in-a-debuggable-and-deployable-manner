# How to reverse open a SharePoint WSP package in a debuggable and deployable manner?
We recently faced a challenge at my workplace regarding a Sharepoint 2010 project. Since Microsoft no longer supports Sharepoint 2010 and we were late in upgrading it, the source codes were missing. To address this, we needed to upgrade it by creating the source code from deployed WSP packages. Unfortunately, I couldn't find a satisfactory solution for this issue on the internet. Most solutions involved opening it as a CAB file, creating a Sharepoint project, opening all DLL files using tools like ILSPY or similar programs for reversing DLL files, and creating each file one by one by copy-pasting their contents. With 14 packages to reverse and a tight deadline, I had to improvise and devised a solution that significantly streamlined the process.
This method allows you to extract the source code from a WSP package within a day. The best part is that you can debug and deploy the same project. Although I haven't encountered any significant problems or losses so far, there might be minor issues. However, since you can debug, identifying and fixing problems becomes relatively straightforward. I hope you never find yourself in a situation where you need to use this method, but if you ever lose the source code or are uncertain about a potential mismatch between the source code and production code, I'll guide you on how to perform this process.
1.	First, we need to obtain the WSP packages. Once you have the packages, we will open them using Visual Studio without converting them into a .cab file.
2.	In Visual Studio, select "New Project," name the solution as desired, and choose the "SharePoint 2016 – Import Solution Package" option. If you cannot find this choice, you may need to go to the Visual Studio Installer and download the necessary packages.  
![2](https://gist.github.com/assets/89383169/b454b851-798f-4967-aad7-e9cdba81aa1e)  
3.	On the new screen, fill in the required fields. In my case, I'll choose "Deploy as a farm solution." Additionally, provide the site URL, although this can be modified later  
![3](https://gist.github.com/assets/89383169/f7ae5258-f5ce-4133-bc94-3aaa5aeee75c)  
4.	On the next screen, it will prompt you for the WSP package. Select the WSP package and proceed to the next step  
![4](https://gist.github.com/assets/89383169/941b0a0f-d8b5-4066-b35e-159a26f11bab)  
5.	During this phase, your project files are prepared quite quickly. However, I must warn you that sometimes the loading screen may not disappear immediately, but that doesn't necessarily mean it's stuck. Simply click on the windows in the background of the loading pop-up screen, and it will close. By default, it usually selects all the files. If not, I recommend choosing all of them. Having extra files doesn't cause any issues, but missing files can. Once you've made your selections, click the "Finish" button.  
![5](https://gist.github.com/assets/89383169/8f083a88-f73e-4de2-b469-ce366109243c)  
6.	After the project creation process is complete, you now have a deployable SharePoint project. For example, mine looks like this.  
![6](https://gist.github.com/assets/89383169/5b9142da-d6f4-47d2-a3dc-dd135e80ee13)  
The downsides include the fact that your project might be a bit messy, and the names could be uncertain. However, with a little effort, you will get used to it. In this project, you will find your CSS, JS, ASCX, WebPart files, etc. However, you won't see any CS files, for example. These backend files are located in the "Other Imported Files" folder. Yet, when you deploy it, backend files still take their place in the "GAC_MSIL" folder. However, to DEBUG it and have full control over the project, we will need these files as well. In the next steps, we will address this.  
7.	Before moving on to the next steps, check your .NET version. If necessary, upgrade it, and make sure you can currently build and deploy this project. If you have more WSP packages to handle, simply right-click on your solution, choose "Add -> New Project," and follow the previous steps. I recommend importing all WSP packages before reversing the DLLs and attaching them (Spoiler Alert).  
8.	In this step, to keep the project organized, I recommend creating two folders named "Internal Libraries" and "External Libraries." The "Internal Libraries" folder will contain reversed DLL packages of the SharePoint project. For instance, if you import "benara.wsp," you will obtain "benara.dll." We will reverse these DLLs and place them into the "Internal Libraries" folder as a Class Library. However, for any other DLLs that you don't need to reverse, simply put them into the "External Libraries" folder and include them in the SharePoint project's Package.package file.  
![8](https://gist.github.com/assets/89383169/80c146a7-fccf-424a-b84f-0a518ea3bd55)  
9.	Now, we will begin reversing our DLLs. I will be using ILSpy and strongly recommend you to use this tool as well. Other reversing tools might not provide a class library that can be easily built. ILSpy is very effective for this purpose. Let's create our workspace by selecting "File -> Manage Assembly Lists." In the opening pop-up window, choose "New," give your list a name, and double-click your list to open it.  
![9](https://gist.github.com/assets/89383169/d9e1b5dc-ef4a-4cfb-b88c-43da4a2439f7)  
10.	First, I will add one DLL that I gathered from the "Other Imported Files" folder.  
![10](https://gist.github.com/assets/89383169/e743fac9-d6af-47eb-8b15-6454a4d1bc19)  
As you can see, it gives me a warning. This DLL has its dependencies, and ILSpy cannot find them. If they are located in the computer's assembly folder, ILSpy can locate them. However, if not, you shouldn't reverse it yet. There are two options: you can see the assembly logs by clicking "Show assembly load log," and by reading it, you can detect missing DLLs and add them to your ILSpy assembly list before reversing the code. In this case, I recommend importing all WSP packages into your Visual Studio solution, as we did in the previous steps. Then, gather all DLL files located in the "bin" and "Other Imported Files" folders and add them all to your ILSpy assembly list. This way, you will have almost all the necessary DLL files. Just be careful about your main DLL; for example, if your WSP package name is "benara.wsp" and you find a "benara.dll" in the "Other Imported Files" folder and the "bin" folder after importing the project, **use the "benara.dll" located in the "Other Imported Files" folder, not the "bin" folder.** There still might be missing libraries; just do the checking and provide them if needed.  
11. Now, as the final step before reversing the code, I recommend changing one setting in ILSpy to avoid potential issues. Go to "View -> Options," scroll to the end of the opening screen, and uncheck the option "Use new SDK format..." as shown in the image below.  
![11](https://gist.github.com/assets/89383169/0d7e58d4-5f43-4eb9-a582-6c6631062cf2)  
Both approaches work, but based on my experience, I encountered some issues that might be fixable, but I believe "This is the way." Now, we are ready to proceed.  
12.	Simply right-click on the project from your list and select "Save the code." Save it somewhere; in my case, I will place it into a folder named after the csproj file. For example, if you are saving "senara.csproj," put it into a folder named "senara" and then place the "senara" folder into the "Internal Libraries" folder within your Visual Studio solution.  
13.	Now, let's add our reversed DLL files project to our solution. If you placed it in the "Internal Libraries" folder, simply right-click on your folder and choose "Add -> Existing Project." Repeat the same process for all other reversed DLL files in your solution. Don't forget, if you reversed it, put it into "Internal"; otherwise, for example, if you have a "Camlex.dll" in the "Other Imported Files" folder, add it to the "External Libraries" folder without reversing.  
14.	Now, we come to the tricky part. Currently, when we deploy our solution, our reversed DLLs are not deploying in the "GAC_MSIL" folder; the old ones are still taking their place there. So, you can't change the code even if you make modifications in "Internal Libraries," and you can't DEBUG it. We need to trick the SharePoint project to build the new DLL into the GAC.
15.	To accomplish this, first, we need to inform our SharePoint project that it can't deploy its own DLL into the "GAC_MSIL" folder. To do this, click on the SharePoint project, and in the properties window, set the "Include Assembly In Package" option to false. After doing this, when you deploy the project, nothing will go into the "GAC_MSIL" folder.  
![15](https://gist.github.com/assets/89383169/5e698e70-8af2-4617-b9d1-ec27237d39a0)  
16.	Now, we must attach our new reversed "Internal Library" DLL into the SharePoint project. To do this, simply go into your SharePoint project's "Package" folder and double-click the Package.package file. Go to the "Advanced" tab and click "Add Project Assembly" in the pop-up window. Then, choose your project located in the "Internal Libraries" folder, and the "Deployment Target" will, of course, be "GlobalAssemblyCache" (Be careful not to choose the SharePoint project if they are named the same. It might help to click and extend the "Source Project" field; the project list is sorted by creation time).  
![16](https://gist.github.com/assets/89383169/7d778182-b2c9-47ee-8d3d-e9c0525291fa)  
![162](https://gist.github.com/assets/89383169/861ad236-d430-4173-85ea-6b287e188219)  
17.	Repeat this step for all other libraries that exist in the "Other Imported Files" folder. Don't forget the DLLs that you put into the "External Libraries." But to add them to the package, this time you should choose "Add Existing Assembly" and select your DLL by adding the "Source Path."  
![17](https://gist.github.com/assets/89383169/2e1cba79-7b48-4733-9c34-02b2cbf87ff9)  
18.	Now, when we deploy our SharePoint Solution, we expect our reversed DLL to take its place in the "GAC_MSIL" folder. Sometimes, it happens flawlessly, but in some cases, like mine, it did not. So, I took precautions by adding a post-build command to the reversed project. I highly recommend you do the same. To do this, right-click your project, select "Properties," go to the "Build Events" tab, and put the command

    ***"%ProgramFiles%\Microsoft SDKs\Windows\v10.0A\bin\NETFX 4.8 Tools\gacutil.exe" /i "$(TargetPath)"***

    into the "Post-build events command line" section. There might be two warnings while doing this: in the command, "" tags are         included, and your "gacutil.exe" location might be different. Adjust the command as needed. Now, we expect no problems with our DLL taking its place in the "GAC_MSIL" folder  
![18](https://gist.github.com/assets/89383169/a0da4a98-7fb6-4935-8ccd-86d9e3565146)  
19.	Another tricky part is that when you try to deploy your SharePoint project, your "Internal Library" project may need to be strongly named, and you might encounter an error about this such as **"Failure adding assembly to the cache: Attempt to install an assembly without a strong name"**. You can easily fix that by right-clicking your "Internal Library" project, going into the "Signing" tab, and choosing the "Sign the assembly" checkbox, creating a new .snk file with any name. However, after that, your project's "PublicKeyToken" will change, and it will cost you a lot of work to fix this. You will need to change your ASCX files  

    **<%@ Assembly Name="Benara.Web, Version=1.0.0.0, Culture=neutral, PublicKeyToken=ea300ab52f6kl1cd" %>**  

    lines according to your new "PublicKeyToken." For those who don't know how to find the newly created key, you can locate it in the _"C:\Windows\Microsoft.NET\assembly\GAC_MSIL{yourprojectname}"_ path. In this path, there should be a folder named as "v4.0_1.0.0.0__ ea300ab52f6kl1cd," and the last part after "_" is your "PublicKeyToken." Other than .ascx files, if you used your working old web.config "SafeControl" lines, you must change the "PublicKeyToken" in there as well (I will get to the web.config topic later on). In the next section, I will talk about how to fix this problem.  
![19](https://gist.github.com/assets/89383169/ac1887cd-d8bb-4654-ac53-4889fdc51a40)  
20.	There is a very easy solution to the "PublicKeyToken" problem. If you have an old source code, even though you can't fully trust it, you can still claim .snk files for all your projects and simply add them to your current "Internal Library" project. Also, after "Import Solution Package" in our newly created SharePoint projects, there should be a .snk file. You can take those and put them into the "Internal Libraries" project and attach them. In the end, you save your "PublicKeyToken" and a lot of time. Let's discuss this with an example to make it clear.  
	First, you imported your "benara.wsp" and got a SharePoint project named "benara," then you reversed your "benara.dll" and obtained your "Internal Library" project. There will be a .snk file in the "benara" SharePoint project, and there won't be a .snk file in the "Internal Library" project. You will simply go to your SharePoint project, right-click and choose "Properties," go to the "Signing" tab, and uncheck the "Sign the assembly" checkbox. Why? Because your "GAC_MSIL" DLL no longer exists here. Copy or cut your .snk file and carry it into the "Internal Library" project (anywhere in the project, but generally .snk files are located in the "Properties" folder; I also prefer there). Go to the "Signing" tab, check "Sign the assembly," and browse the .snk file that you just carried. It's done. Now your "GAC_MSIL" DLL will be created with the correct "PublicKeyToken." Now you can deploy your SharePoint project, and everything will be okey.  
![20](https://gist.github.com/assets/89383169/c91d0cce-9907-441f-b7ab-fa4a668e866c)  
21.	One last warning, and this is one of the downsides as well: by this method, web.config "Safe Control" lines won't be created automatically. So, if you are working in another environment, you will get an error when trying to reach the webpage. For example, a web part named "blabla.webpart" might show an error stating it "does not exist or is not registered as safe." In the best scenario, if this application is running on some server, just obtain the "Safe Control" lines and add them to your new environment's web.config file. If you can't access those, you'll need to create them manually, but I hope this is not your case.  
22.	When you follow these steps for all .wsp packages and necessary .dll files, you will have recovered your source code. You can deploy it and also debug it by simply using "attach to process." The architecture might be a bit messy, but you can easily locate any files you need and proceed with your work. However, for adding something new, creating a new SharePoint project as you desire and deploying it to the farm might be the best practice since it won't be messy.  
    I hope this method works for those who need this kind of reversing, but I sincerely hope that nobody needs it. If you have any further questions or need additional assistance, feel free to ask.  









