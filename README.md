---


---

<h1 id="runbook-for-high-available-kubernetes-cluster">Runbook for High Available Kubernetes Cluster</h1>
<p><strong>Note: [IMP] Document preparation in progress and not complete.</strong></p>
<h2 id="assumption">Assumption:</h2>
<p><em>There is a existing on-premises high available Kubernetes cluster.</em></p>
<h2 id="basic-kubernetes-commands">Basic Kubernetes Commands:</h2>
<ul>
<li>To check the version of the kubernetes cluster:</li>
</ul>
<pre><code>    kubeadm version
</code></pre>
<ul>
<li>To check the nodes existing in the kubernetes cluster:</li>
</ul>
<pre><code>    kubectl get nodes
</code></pre>
<ul>
<li>To display the addresses of the master and services in kubernetes cluster:</li>
</ul>
<pre><code>    kubectl cluster-info
</code></pre>
<ul>
<li>To further debug and diagnose cluster problems , use:</li>
</ul>
<pre><code>    kubectl cluster-info dump
</code></pre>
<ul>
<li>To mark a node as un-scheduled:</li>
</ul>
<pre><code>    kubectl cordon &lt;node-name&gt;
</code></pre>
<ul>
<li>To mark a node as scheduled:</li>
</ul>
<pre><code>    kubectl uncordon &lt;node-name&gt;
</code></pre>
<ul>
<li>To bring a node out of cluster and maintain it , Use drain:</li>
</ul>
<pre><code>    kubectl drain &lt;node-name&gt;
</code></pre>
<h2 id="maintenance-activities">Maintenance Activities</h2>
<h3 id="to-add-a-new-worker-node-to-a-existing-kubernetes-cluster">1. To add a new worker node to a existing kubernetes cluster:</h3>
<ul>
<li>
<p>Verify the below system requirements are met<br>
<a href="https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin">https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin</a></p>
</li>
<li>
<p>Pre-requisites:<br>
-Install Docker ( Container runtime )<br>
-Install Kubectl<br>
-Install Kubelet<br>
-Install Kubeadm</p>
</li>
<li>
<p>References:<br>
<a href="https://docs.docker.com/engine/install/">https://docs.docker.com/engine/install/</a><br>
<a href="https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl">https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl</a></p>
</li>
</ul>
<p><strong>Steps:</strong></p>
<p>SSH to kubernetes master and execute the following steps.</p>
<ol>
<li>Get the token from the Kubernetes master.</li>
</ol>
<pre><code>    kubeadm tokens list
</code></pre>
<ol start="2">
<li>If your cluster was initialized over 24-hour ago, the list will likely be empty, since a token’s lifespan is only 24-hours. Then create a new token</li>
</ol>
<pre><code>    kubeadm token create --print-join-command
</code></pre>
<ol start="3">
<li>Use kubeadm to list all tokens in order to verify our new one.</li>
</ol>
<pre><code>    kubeadm tokens list
</code></pre>
<p>SSH to Worker node which you need to add and execute the following tasks.</p>
<ol start="4">
<li>Use the kubeadm join command with our new token to join the node to our cluster</li>
</ol>
<pre class=" language-shell"><code class="prism  language-shell">kubeadm join &lt;api-server-endpoint&gt; --discovery-token abcdef.1234567890abcdef --discovery-token-ca-cert-hash sha256:1234..cdef 1.2.3.4:6443
</code></pre>
<p>Example:</p>
<pre><code>kubeadm join 192.168.1.130:6443 --token qt57zu.wuvqh64un13trr7x --discovery-token-ca-cert-hash sha256:5ad014cad868fdfe9388d5b33796cf40fc1e8c2b3dccaebff0b066a0532e8723
</code></pre>
<ol start="5">
<li>Verify the status of the node that is added to ensure it is successfully added without any issues.</li>
</ol>
<pre><code>    kubectl get nodes &lt;node-name&gt;
</code></pre>
<h3 id="to-remove-a-worker-node-gracefully-from-a-kubernetes-cluster">2. To remove a worker node gracefully from a kubernetes cluster</h3>
<p><strong>Steps:</strong></p>
<ol>
<li>List all the nodes in the existing cluster</li>
</ol>
<pre><code>kubectl get nodes
</code></pre>
<ol start="2">
<li>Drain the node that needs to be removed.</li>
</ol>
<pre><code>    kubectl drain &lt;node-name&gt;
</code></pre>
<p>You can use kubectl drain to safely evict all of your pods from a node before you perform maintenance on the node (e.g. kernel upgrade, hardware maintenance, etc.). Safe evictions allow the pod’s containers to gracefully terminate.</p>
<ol start="3">
<li>Incase you have daemon-sets and  that are running in your cluster and local volumes. You can use the following options. It is always recommended not  to use the local host filesystems for the persistent volumes.</li>
</ol>
<pre><code>kubectl drain &lt;node-name&gt; --ignore-daemonsets --delete-local-data
</code></pre>
<ol start="4">
<li>
<p>Perform the maintenance work such as kernel upgrade, hardware upgrade, etc.</p>
</li>
<li>
<p>Enable the node back to receive the load of the kubernetes cluster and make it scheduled.</p>
</li>
</ol>
<pre><code>kubectl uncordon &lt;node-name&gt;
</code></pre>
<p>Incase , if you don’t want the node to be part of the existing cluster and want to remove it permanently.  Instead of step 5 , Please execute step 6</p>
<ol start="6">
<li>Delete the node from the cluster and perform a kubeadm reset.</li>
</ol>
<pre><code>kubectl delete node &lt;node-name&gt;
</code></pre>
<pre><code>kubeadm reset ( Make sure it is executed on the removed worker node.)
</code></pre>

