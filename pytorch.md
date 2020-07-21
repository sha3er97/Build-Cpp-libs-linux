---


---

<h1 id="pushpin--building-pytorch-c-for-linux">📌  Building PyTorch C++ for linux!</h1>
<h3 id="books-used-resources-">📚 Used resources :</h3>
<ul>
<li><a href="https://pytorch.org/tutorials/advanced/cpp_export.html#step-1-converting-your-pytorch-model-to-torch-script">Loading a TorchScript Model in C++</a></li>
<li><a href="https://github.com/pytorch/pytorch">pytorch Repo on Github</a></li>
</ul>
<h3 id="steps-">Steps :</h3>
<p><strong>1.</strong> install anaconda environment from <a href="https://www.anaconda.com/distribution/#download-section">Anaconda</a> or by terminal</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">wget</span> https://repo.anaconda.com/archive/Anaconda3-2020.02-Linux-x86_64.sh
</code></pre>
<p>-&gt; here I chose the <a href="https://repo.anaconda.com/archive/Anaconda3-2020.02-Linux-x86_64.sh">64-Bit (x86) Installer</a><br>
<strong>2.</strong> <code>Optional</code> if you don’t have them already installed Pytorch documentation advices to download the following<br>
If you want to compile with CUDA support, install :</p>
<ul>
<li><a href="https://developer.nvidia.com/cuda-downloads">NVIDIA CUDA</a>  9.2 or above</li>
<li><a href="https://developer.nvidia.com/cudnn">NVIDIA cuDNN</a>  v7 or above</li>
<li><a href="https://gist.github.com/ax3l/9489132">Compiler</a>  compatible with CUDA</li>
</ul>
<p><strong>3.</strong> Install Dependencies</p>
<pre class=" language-bash"><code class="prism  language-bash">conda <span class="token function">install</span> numpy ninja pyyaml mkl mkl-include setuptools cmake cffi
conda <span class="token function">install</span> -c pytorch magma-cuda102  <span class="token comment"># or [ magma-cuda101 | magma-cuda100 | magma-cuda92 ] depending on your cuda version</span>
</code></pre>
<p><strong>4.</strong> Get the PyTorch Source</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">git</span> clone --recursive https://github.com/pytorch/pytorch
<span class="token function">cd</span> pytorch
<span class="token function">git</span> submodule <span class="token function">sync</span>
<span class="token function">git</span> submodule update --init --recursive
</code></pre>
<p><strong>5.</strong> Install PyTorch</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">export</span> CMAKE_PREFIX_PATH<span class="token operator">=</span>$<span class="token punctuation">{</span>CONDA_PREFIX:-<span class="token string">"<span class="token variable"><span class="token variable">$(</span><span class="token function">dirname</span> <span class="token punctuation">$(</span>which conda<span class="token variable">)</span></span>)/../"</span><span class="token punctuation">}</span>
python setup.py <span class="token function">install</span>
</code></pre>
<h3 id="now-pytorch-is-installed-on-your-device">now pytorch is installed on your device</h3>
<hr>
<h3 id="you-may-want-to-use-pytorch-on-python-models-..-here-is-the-easy-way-">you may want to use pytorch on python models … here is the easy way :</h3>
<p><strong>1.</strong> get your model ready for converting it to a <a href="https://pytorch.org/docs/master/jit.html">Torch Script</a><br>
by <a href="https://pytorch.org/tutorials/advanced/cpp_export.html#converting-to-torch-script-via-tracing">tracing</a> or <a href="https://pytorch.org/tutorials/advanced/cpp_export.html#converting-to-torch-script-via-annotation">annotating</a><br>
we will take a tracing example here :</p>
<pre class=" language-python"><code class="prism  language-python"><span class="token keyword">import</span> torch
<span class="token keyword">import</span> torchvision

<span class="token comment"># An instance of your model.</span>
model <span class="token operator">=</span> torchvision<span class="token punctuation">.</span>models<span class="token punctuation">.</span>resnet18<span class="token punctuation">(</span><span class="token punctuation">)</span>

<span class="token comment"># An example input you would normally provide to your model's forward() method.</span>
example <span class="token operator">=</span> torch<span class="token punctuation">.</span>rand<span class="token punctuation">(</span><span class="token number">1</span><span class="token punctuation">,</span> <span class="token number">3</span><span class="token punctuation">,</span> <span class="token number">224</span><span class="token punctuation">,</span> <span class="token number">224</span><span class="token punctuation">)</span>

<span class="token comment"># Use torch.jit.trace to generate a torch.jit.ScriptModule via tracing.</span>
traced_script_module <span class="token operator">=</span> torch<span class="token punctuation">.</span>jit<span class="token punctuation">.</span>trace<span class="token punctuation">(</span>model<span class="token punctuation">,</span> example<span class="token punctuation">)</span>
<span class="token comment">#save the model in a file script</span>
traced_script_module<span class="token punctuation">.</span>save<span class="token punctuation">(</span><span class="token string">"traced_resnet_model.pt"</span><span class="token punctuation">)</span>
</code></pre>
<p>you may face an error here in <code>import torchvision</code><br>
you solve it by installing it firstly :</p>
<pre class=" language-bash"><code class="prism  language-bash">pip <span class="token function">install</span> -U torchvision<span class="token operator">==</span>0.4.0
</code></pre>
<p><strong>2.</strong> A Minimal C++ Application</p>
<pre class=" language-c"><code class="prism ++ language-c"><span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;torch/script.h&gt;</span> </span><span class="token comment">// One-stop header.</span>

<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;iostream&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;memory&gt;</span></span>

<span class="token keyword">int</span> <span class="token function">main</span><span class="token punctuation">(</span><span class="token keyword">int</span> argc<span class="token punctuation">,</span> <span class="token keyword">const</span> <span class="token keyword">char</span><span class="token operator">*</span> argv<span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  <span class="token keyword">if</span> <span class="token punctuation">(</span>argc <span class="token operator">!=</span> <span class="token number">2</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    std<span class="token punctuation">:</span><span class="token punctuation">:</span>cerr <span class="token operator">&lt;&lt;</span> <span class="token string">"usage: example-app &lt;path-to-exported-script-module&gt;\n"</span><span class="token punctuation">;</span>
    <span class="token keyword">return</span> <span class="token operator">-</span><span class="token number">1</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>

  torch<span class="token punctuation">:</span><span class="token punctuation">:</span>jit<span class="token punctuation">:</span><span class="token punctuation">:</span>script<span class="token punctuation">:</span><span class="token punctuation">:</span>Module module<span class="token punctuation">;</span>
  try <span class="token punctuation">{</span>
    <span class="token comment">// Deserialize the ScriptModule from a file using torch::jit::load().</span>
    module <span class="token operator">=</span> torch<span class="token punctuation">:</span><span class="token punctuation">:</span>jit<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">load</span><span class="token punctuation">(</span>argv<span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
  <span class="token function">catch</span> <span class="token punctuation">(</span><span class="token keyword">const</span> c10<span class="token punctuation">:</span><span class="token punctuation">:</span>Error<span class="token operator">&amp;</span> e<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    std<span class="token punctuation">:</span><span class="token punctuation">:</span>cerr <span class="token operator">&lt;&lt;</span> <span class="token string">"error loading the model\n"</span><span class="token punctuation">;</span>
    <span class="token keyword">return</span> <span class="token operator">-</span><span class="token number">1</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>

  std<span class="token punctuation">:</span><span class="token punctuation">:</span>cout <span class="token operator">&lt;&lt;</span> <span class="token string">"ok\n"</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<p><strong>3.</strong> compile c++ file using cmake<br>
create <code>CMakeLists.txt</code> file in same directory of C++ file which we will name here <code>example-app.cpp</code></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">nano</span> CMakeLists.txt
</code></pre>
<p>write the following :</p>
<pre class=" language-c"><code class="prism ++ language-c"><span class="token function">cmake_minimum_required</span><span class="token punctuation">(</span>VERSION <span class="token number">3.0</span> FATAL_ERROR<span class="token punctuation">)</span>
<span class="token function">project</span><span class="token punctuation">(</span>custom_ops<span class="token punctuation">)</span>

<span class="token function">find_package</span><span class="token punctuation">(</span>Torch REQUIRED<span class="token punctuation">)</span>

<span class="token function">add_executable</span><span class="token punctuation">(</span>example<span class="token operator">-</span>app example<span class="token operator">-</span>app<span class="token punctuation">.</span>cpp<span class="token punctuation">)</span>
<span class="token function">target_link_libraries</span><span class="token punctuation">(</span>example<span class="token operator">-</span>app <span class="token string">"${TORCH_LIBRARIES}"</span><span class="token punctuation">)</span>
<span class="token function">set_property</span><span class="token punctuation">(</span>TARGET example<span class="token operator">-</span>app PROPERTY CXX_STANDARD <span class="token number">14</span><span class="token punctuation">)</span>
</code></pre>
<p>now the directory should be something like this</p>
<pre><code>example-app/
  CMakeLists.txt
  example-app.cpp
</code></pre>
<p><strong>4.</strong> compile cpp file using cmake<br>
-&gt; remember where you installed the <code>libtorch</code> folder becuse you will need it right now</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">mkdir</span> build
<span class="token function">cd</span> build
cmake -DCMAKE_PREFIX_PATH<span class="token operator">=</span><span class="token variable"><span class="token variable">`</span>path to libtorch folder<span class="token variable">`</span></span> <span class="token punctuation">..</span>
cmake --build <span class="token keyword">.</span> --config Release
</code></pre>
<p>after finishing making …</p>
<p><strong>5.</strong> run executable file<br>
you will find it in the <code>build</code> folder you just created</p>
<pre class=" language-bash"><code class="prism  language-bash">./example-app <span class="token variable"><span class="token variable">`</span><span class="token operator">&lt;</span>path_to_model<span class="token operator">&gt;</span><span class="token variable">`</span></span>/traced_resnet_model.pt
</code></pre>
<p>in case every thing is okay you will expect as simple <code>ok</code> in the terminal</p>

