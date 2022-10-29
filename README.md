# Patching ETW 

Event Tracing for Windows (ETW) provides a mechanism to trace and log events that are raised by user-mode applications and kernel-mode drivers.  
ntdll!EtwEventWrite is responsible for writing an event , it's not actually the function that do the Event Writing job, by reversing it, we can see that it calls ntdll!EtwpEventWriteFull that do the actual Event Writing :  
![IDAetw](https://user-images.githubusercontent.com/110354855/198856050-d5a3d460-0ad7-494b-abb8-018333d2f497.png)



