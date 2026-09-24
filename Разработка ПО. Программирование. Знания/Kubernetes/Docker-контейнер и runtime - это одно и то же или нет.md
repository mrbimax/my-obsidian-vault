Есть Docker как стандарт, по которому описываются контейнеры, а есть Docker-движок, он же runtime, — это то, что запускает контейнер.  
  
В Kubernetes благодаря [Container Runtime Interface (CRI)](https://kubernetes.io/docs/concepts/architecture/cri/) API в контейнерах можно запускать разные runtime, например CRI-O, Containerd.  
  
Так как Docker-движок старше, чем Kubernetes, он не отвечает стандартам CRI, поэтому уже некоторое время [Docker runtime не поддерживается в Kubernetes](https://kubernetes.io/blog/2022/02/17/dockershim-faq/).  
  
Но это [не означает](https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/), что сами Docker-контейнеры нельзя использовать в Kubernetes.  
  
За словом Docker действительно скрывается много всего, что вызывает путаницу. Об этом даже говорил сам Джо Беда, один из создателей Kubernetes. Так что вы точно не одиноки :)  
  
![image](https://habrastorage.org/r/w1560/webt/xs/zq/wb/xszqwbod7z4oqtdy_mz_8izmzxm.png)  
  
Подробнее о том, как соотносятся контейнеры, Container Runtime, CRI и о судьбе Docker runtime в Kubernetes, можно почитать [тут](https://www.tutorialworks.com/difference-docker-containerd-runc-crio-oci/).