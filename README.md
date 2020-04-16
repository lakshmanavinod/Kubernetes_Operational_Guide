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
<h3 id="upgrading-the-kubernetes-version">3. Upgrading the kubernetes version:</h3>
<p><strong>Basic Workflow:</strong></p>
<ul>
<li>Upgrade the primary control plane node.</li>
<li>Upgrade additional control plane nodes. ( Other Kubernetes masters )</li>
<li>Upgrade worker nodes</li>
</ul>
<p><strong>Pre-requisites:</strong></p>
<ul>
<li>Swap must be disabled. To disable , Click <a href="https://serverfault.com/questions/684771/best-way-to-disable-swap-in-linux">here</a></li>
</ul>
<p><strong>Few things to note</strong>:</p>
<ul>
<li>After upgrade, all the containers are restarted.</li>
<li>The upgrade can be only done for the next immediate minor version . For example, we can upgrade kubeadm from 1.18.y to 1.18.y+1 but not 1.18.y to 1.18.y+2</li>
</ul>
<p><strong>Steps:</strong><br>
Let’s take Centos/RHEL machine for this activity, If you have a different operating system , Please refer the here.</p>
<ol>
<li>
<p>Query the latest stable version of 1.18.</p>
<pre><code>yum list --showduplicates kubeadm --disableexcludes=kubernetes
</code></pre>
</li>
<li>
<p>Upgrade control plane ( Kubernetes master which acts as leader )</p>
<pre><code>yum install -y kubeadm-1.18.&lt;latest_version&gt; --disableexcludes=kubernetes
</code></pre>
</li>
<li>
<p>SSH to Kubernetes master and upgrade kudeadm version</p>
<pre><code>kubeadm version
</code></pre>
</li>
<li>
<p>Drain the kubernetes master node</p>
<pre><code>kubectl drain &lt;node-name&gt; --ignore-daemonsets
</code></pre>
</li>
<li>
<p>Upgrade the kubernetes master node</p>
<pre class=" language-shell"><code class="prism  language-shell">sudo kubeadm upgrade plan
</code></pre>
</li>
<li>
<p>Manually upgrade CNI Plugin , if required . Please refer <a href="https://kubernetes.io/docs/concepts/cluster-administration/addons/">addons</a> for details.</p>
</li>
<li>
<p>Uncordon the kubernetes master</p>
<pre><code>kubectl uncordon &lt;cp-node-name&gt;
</code></pre>
</li>
<li>
<p>Upgrade other kubernetes masters,follow the same steps but instead of:</p>
<pre><code>sudo kubeadm upgrade node
</code></pre>
<p>use:</p>
<pre><code>sudo kubeadm upgrade apply
</code></pre>
<p>Also, <code>sudo kubeadm upgrade plan</code> is not needed.</p>
</li>
<li>
<p>Upgrade kubelet and kubectl on all Kuberenetes master nodes.</p>
<pre><code>yum install -y kubelet-1.18.x-0 kubectl-1.18.x-0 -   -disableexcludes=kubernetes

sudo systemctl restart kubelet
</code></pre>
</li>
<li>
<p>Upgrade kudeadm on all the worker nodes</p>
<pre><code> yum install -y kubeadm-1.18.&lt;latest_version&gt; --disableexcludes=kubernetes
</code></pre>
</li>
<li>
<p>Prepare the nodes for maintenance , drain all the nodes from cluster.</p>
<pre><code>kubectl drain &lt;node-to-drain&gt; --ignore-daemonsets
</code></pre>
</li>
<li>
<p>Upgrade the kubeadm configuration on all worker nodes.</p>
<pre><code>sudo kubeadm upgrade node
</code></pre>
</li>
<li>
<p>Upgrade kubelet and kubectl on all the worker nodes.</p>
<pre><code> yum install -y kubelet-1.18.x-0 kubectl-1.18.x-0 --disableexcludes=kubernetes
 sudo systemctl restart kubelet
</code></pre>
</li>
<li>
<p>Uncordon all the nodes and bring it back to schedulable.</p>
<pre><code>kubectl uncordon &lt;node-to-drain&gt;
</code></pre>
</li>
<li>
<p>Verify the status of the cluster.</p>
<pre><code>kubectl get nodes
</code></pre>
</li>
</ol>
<p>Incase , if you the upgrade fails, there would be automatic rollback that happens. For more instructions or details , Please refer to <a href="https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/">https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/</a></p>
<h3 id="switch-between-namespaces-and-multiple-clusters">4. Switch between namespaces and multiple clusters:</h3>
<h4 id="a-to-switch-between-different-namespaces-in-a-kubernetes-cluster">4.a To switch between different namespaces in a kubernetes cluster:</h4>
<p>When you want to interact with objects in a different namespace than “default”, you must pass the <code>-n</code> flag to <code>kubectl</code>. For example, to see the pods in the Kubernetes system namespace, <code>kube-system</code>, we can run the following:</p>
<pre><code>kubectl get pods -n kube-system
</code></pre>
<p>What if , If we want to operate on a particular namespace other than “default” for full day. Following is how we can switch the namespace in our context.</p>
<p><strong>Steps:</strong></p>
<ol>
<li>To view the current namespace:</li>
</ol>
<pre><code>kubectl config view | grep namespace
</code></pre>
<ol start="2">
<li>To change the namespace in the current-context</li>
</ol>
<pre><code>kubectl config set-context --current --namespace=my-namespace
</code></pre>
<ol start="3">
<li>Verify the namespace has been udpated in the current context</li>
</ol>
<pre><code>kubectl config view | grep namespace
</code></pre>
<h4 id="b-to-switch-between-different-kubernetes-clusters-using-kubectl.">4.b To switch between different kubernetes clusters using kubectl.</h4>
<p><strong>Steps:</strong></p>
<ol>
<li>KUBECONFIG environment variable plays an important role. It holds the list of kubeconfig files.For Linux and Mac, the list is colon-delimited.</li>
</ol>
<p>Gather all your different Kube-conf files in a folder. And export these files separated by a colon. Also, you can add to your ~/.bash_profile file.</p>
<pre><code>export KUBECONFIG=&lt;kube-conf-file1&gt;:&lt;kube-conf-file2&gt;:..so..on
</code></pre>
<ol start="2">
<li>Verify all the contexts are set by getting the contexts. It should list all the contexts and names, namespaces, etc.</li>
</ol>
<pre><code>kubectl config get-contexts
</code></pre>
<ol start="3">
<li>Set the context and start using it.</li>
</ol>
<pre><code> kubectl config use-context &lt;context-Name&gt;
</code></pre>
<ol start="4">
<li>Verify the currently loaded context and make sure it points to the right kubernetes cluster.</li>
</ol>
<pre><code>kubectl config current-context 
</code></pre>

