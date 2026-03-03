
- addWorker 함수에서 일이 일어남 
그때 thread pool executor의 largestPoolSize 라는것도 업데이트 될 수 있음.
Worker.thread는 thread factory가 만드는데, 이때 별 설정 없으면 
java/util/concurrent/Executors.java 밑에 있는 DefaultThreadFactory 를 사용함.
얘가 스레드를 만들때마다 threadNumber는 걍 monotonic increase함. 즉 스레드 풀에 스레드가 죽어 없어져도 새로 만드는 애는 번호가 계속 증가함.

스레드는 언제 없어지는가?
runWorker하고 나서 finally 구문에서 잡아서 processWorkerExit 함수에서 없애줌.


