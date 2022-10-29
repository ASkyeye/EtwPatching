# Patching ETW 

### Event Tracing for Windows (ETW) provides a mechanism to trace and log events that are raised by user-mode applications and kernel-mode drivers.  
### ntdll!EtwEventWrite is responsible for writing an event , it's not actually the function that do the Event Writing job, by reversing it, we can see that it calls ntdll!EtwpEventWriteFull that do the actual Event Writing :  
  
![IDAetw](https://user-images.githubusercontent.com/110354855/198856050-d5a3d460-0ad7-494b-abb8-018333d2f497.png)  
   
### this call is done at offset (0x214 -  0x1f0 = 0x24) and it takes 5 bytes, the idea here is to write a program that identify 0xe8 the opcode of the call instruction and overwrite this call memory with 5 bytes , so that the call will never done  
  

![TheCall](https://user-images.githubusercontent.com/110354855/198856198-a66c74e6-5833-47d4-a82c-1646206bf61b.png)




