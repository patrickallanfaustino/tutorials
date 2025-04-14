# Workflow Install Gromacs 2025.x com ROCm 6.3.3 e AdaptiveCpp 24.x no Ubuntu 24.04 Noble Numbat

![GitHub repo size](https://img.shields.io/github/repo-size/patrickallanfaustino/tutorials?style=for-the-badge)
![GitHub language count](https://img.shields.io/github/languages/count/patrickallanfaustino/tutorials?style=for-the-badge)
![GitHub forks](https://img.shields.io/github/forks/patrickallanfaustino/tutorials?style=for-the-badge)
![GitHub Release](https://img.shields.io/github/v/release/patrickallanfaustino/tutorials?style=for-the-badge)
![GitHub Tag](https://img.shields.io/github/v/tag/patrickallanfaustino/tutorials?style=for-the-badge)

<img src="imagem1.png" alt="computer">

> Tutorial para compilar o Gromacs 2025.x com suporte NNPOT-PyTorch (Redes Neurais), usando AdaptiveCpp 24.xx em backend e ROCm 6.3.3 no Ubuntu 24.04 Kernel 6.11, para utilizar aceleração GPU AMD RDNA2 em desktop.

## 💻 Computador testado e Pré-requisitos:
- CPU Ryzen 9 5900XT, Memória 2x16 GB DDR4, Chipset X470, GPU ASRock RX 6600 CLD 8 GB, dual boot com Windows 11 e Ubuntu 24.04 instalados em SSD's separados.

Antes de começar, verifique se você atendeu aos seguintes requisitos:

- Você tem uma máquina `Linux Ubuntu 24.04` com instalação limpa e atualizado.
- Você tem uma GPU série `AMD RX 6xxx RDNA2`. Não testado com outras arquiteturas.
- Documentações [ROCm 6.3.3](https://rocm.docs.amd.com/projects/install-on-linux/en/docs-6.3.3/index.html), [AdaptiveCpp 24.xx](https://github.com/AdaptiveCpp/AdaptiveCpp).

Você também vai precisar atualizar e instalar pacotes em sua máquina:

```
sudo apt update && sudo apt upgrade
```

```
sudo apt autoremove && sudo apt autoclean
```

Para adicionar ferramentas necessárias ou atualizar com versões mais recentes:

```
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
```
```
sudo apt update && sudo apt upgrade
```
Verifique também a versão do kernel:
```
uname -r
```

---
## 🔧 Instalando Timeshif

O [Timeshift](https://www.edivaldobrito.com.br/como-instalar-o-timeshift-no-ubuntu-linux-e-derivados/) é um software para criar backups. Recomendamos que seja criada backups para cada etapa. Para instalar o `Timeshift`, siga estas etapas:

```
sudo add-apt-repository ppa:teejee2008/timeshift
```
```
sudo apt update
```
```
sudo apt install timeshift
```

>[!TIP]
>
>Se desejar, pode-se instalar o [GRUB CUSTOMIZER](https://www.edivaldobrito.com.br/grub-customizer-no-ubuntu/) para gerenciar o inicializador e [MAINLINE](https://www.edivaldobrito.com.br/como-instalar-o-ubuntu-mainline-kernel-installer-no-ubuntu-e-derivados/) para gerenciar kernel instalados.
>
>```
>sudo add-apt-repository ppa:danielrichter2007/grub-customizer
>sudo apt update
>sudo apt install grub-customizer
>```
>
>```
>sudo add-apt-repository ppa:cappelikan/ppa
>sudo apt update
>sudo apt install mainline
>```
---
## 🔎 Instalando ROCm 5.7.1

Vamos instalar o `ROCm 5.7.1`. Precisamos dar previlégios ao usuário e adicioná-lo a grupos:

```
sudo usermod -a -G render,video $LOGNAME
```
```
echo ‘ADD_EXTRA_GROUPS=1’ | sudo tee -a /etc/adduser.conf
```
```
echo ‘EXTRA_GROUPS=video’ | sudo tee -a /etc/adduser.conf
```
```
echo ‘EXTRA_GROUPS=render’ | sudo tee -a /etc/adduser.conf
```

Download e instalação do pacote `ROCm 5.7.1`:

```
wget https://repo.radeon.com/amdgpu-install/5.7.1/ubuntu/jammy/amdgpu-install_5.7.50701-1_all.deb
```
```
sudo apt install ./amdgpu-install_5.7.50701-1_all.deb
```

Utilizando o `amdgpu-install`, instalar o pacote `rocm,hip,hiplibsdk`:

```
sudo amdgpu-install --usecase=rocm,rocmdev,hip,hiplibsdk
```

Atualizar todos os índices e links de bibliotecas:

```
sudo ldconfig
```

Para verificar a instalação, utilize:

```
sudo clinfo
```
```
sudo rocminfo
```

Pode ser necessário a instalação da biblioteca `rocm-llvm-dev`:
```
sudo apt install rocm-llvm-dev
```

A GPU deverá ser identificada. Caso não consiga, experimente `reboot` e verifique novamente. Instalação ficará em `PATH=/opt/rocm`.

>[!TIP]
>
>Utilize o comando abaixo para listar todos os `cases` disponíveis no `amdgpu-install`:
>
>```
>sudo amdgpu-install --list-usecase
>```
>
>Para remover `amdgpu-install`, utilize:
>
>```
>amdgpu-uninstall
>```
>```
>sudo apt purge amdgpu-install
>```
>
---
## ⌚ Instalando LACT

O aplicativo `LACT` é utilizado para controlar e realizar overclocking em GPU AMD, Intel e Nvidia em sistemas Linux.

```
wget https://github.com/ilya-zlobintsev/LACT/releases/download/v0.7.0/lact-0.7.0-0.amd64.ubuntu-2204.deb
```

>[!TIP]
>
>Faça o download do pacote do [LACT](https://github.com/ilya-zlobintsev/LACT/releases/) de acordo com a distribuição do Linux.
>

```
sudo dpkg -i lact-0.7.0-0.amd64.ubuntu-2204.deb
```
```
sudo systemctl enable --now lactd
```
---
## 🔨 Instalando LLVM e bibliotecas

O `AdaptiveCpp` requer LLVM/Clang e algumas bibliotecas. Para instalar, faça:

```
wget https://apt.llvm.org/llvm.sh
```
```
sudo chmod +x llvm.sh
```
```
sudo ./llvm.sh 16
```
```
sudo apt install -y libclang-16-dev clang-tools-16 libomp-16-dev llvm-16-dev lld-16
```
---
## 📊 Instalando AdaptiveCpp 24.xx

O `AdaptiveCpp 24.xx` irá trabalhar em backend com `ROCm 5.7.1`. Ele contém o `SyCL`. Para instalar:

```
git clone https://github.com/AdaptiveCpp/AdaptiveCpp
```
```
cd AdaptiveCpp
```
```
sudo mkdir build && cd build
```

Para compilar com CMake:

```
sudo cmake .. -DCMAKE_INSTALL_PREFIX=/home/patrick/sycl -DCMAKE_C_COMPILER=/opt/rocm/llvm/bin/clang -DCMAKE_CXX_COMPILER=/opt/rocm/llvm/bin/clang++ -DLLVM_DIR=/opt/rocm/llvm/lib/cmake/llvm/ -DROCM_PATH=/opt/rocm -DWITH_ROCM_BACKEND=ON -DWITH_SSCP_COMPILER=OFF -DWITH_OPENCL_BACKEND=OFF -DWITH_LEVEL_ZERO_BACKEND=OFF -DDEFAULT_TARGETS='hip:gfx1032'
```
```
sudo make install -j16
```

>[!NOTE]
>
>**Meu Caso**: Recomendo criar pastas para as compilações, assim se algo der errado é só apagar e recomeçar. Criei a pasta `sycl` com `sudo mkdir sycl` e indiquei com `-DCMAKE_INSTALL_PREFIX` ao compilar. Em `-DDEFAULT_TARGETS` completar `ABC` em `hip:gfx1ABC` com a informação obtida em `clinfo` ou `rocminfo`. Esse código corresponde ao endereçamento físico da GPU. No `sudo make install -j 16`, a tag `-j 16` define a quantidade de CPUs (16) utilizadas na compilação.

>[!WARNING]
>
>Sempre fique atento aos caminhos de endereçamentos, *i.e* `/path/to/user/...`, porque são os maiores causadores de erros durante as compilações.
---
## 💎 Instalação do Gromacs 2025.x

**OPCIONAL!** Antes de instalar o Gromacs, você talvez queira instalar algumas bibliotecas que melhora o desempenho e eficiência de cálculos no Gromacs. *Essas bibliotecas são opcionais porque o Gromacs já tem BLAS e LAPACK built-in*. No caso abaixo, irá instalar as bibliotecas `BLAS LAPACK 64bit` em `/usr/lib/x86_64-linux-gnu/blas64/libblas64.so` e `/usr/lib/x86_64-linux-gnu/lapack64/liblapack64.so`.

```
sudo apt install liblapack64-dev libblas64-dev
```

**LIBTORCH!** Também é possivel instalar a biblioteca [libtorch](https://pytorch.org/) para utilizar Redes Neurais. Verifique a versão mais recente.

```
wget https://download.pytorch.org/libtorch/cpu/libtorch-cxx11-abi-shared-with-deps-2.6.0%2Bcpu.zip
```
```
unzip libtorch-cxx11-abi-shared-with-deps-2.6.0%2Bcpu.zip
```

**ROCBLAS e ROCSOLVER!** São bibliotecas otimizadas para hardwares AMD. São opcionais e também tem `HIPBLAS HIPSOLVER`. São instaladas com o `amdgpu-install`.

A partir de agora, você poderá seguir a documentação [guia de instalação](https://manual.gromacs.org/current/install-guide/index.html) do Gromacs. No momento de compilar com CMake, utilize:

```
sudo cmake .. -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=ON -DGMX_HWLOC=ON -DCMAKE_C_COMPILER=/opt/rocm/llvm/bin/clang -DCMAKE_CXX_COMPILER=/opt/rocm/llvm/bin/clang++ -DHIPSYCL_TARGETS='hip:gfx1032' -DGMX_GPU=SYCL -DGMX_SYCL=ACPP -DCMAKE_INSTALL_PREFIX=/home/patrick/gromacs20250 -DCMAKE_PREFIX_PATH="/home/patrick/sycl;/home/patrick/libtorch" -DSYCL_CXX_FLAGS_EXTRA=-DHIPSYCL_ALLOW_INSTANT_SUBMISSION=1 -DGMX_EXTERNAL_BLAS=ON -DGMX_EXTERNAL_LAPACK=ON -DGMX_BLAS_USER=/opt/rocm/rocblas/lib/librocblas.so -DGMX_LAPACK_USER=/opt/rocm/rocsolver/lib/librocsolver.so -DGMX_USE_PLUMED=ON -DGMX_NNPOT=TORCH
```
Note que criei uma pasta chamada `gromacs20250` para os arquivos compilados e indiquei com `-DCMAKE_INSTALL_PREFIX`. 

>[!NOTE]
>
>**Meu Caso**: Utilizei as bibliotecas `ROCBLAS` e `ROCSOLVER` para os cálculos, indicando com `-DGMX_EXTERNAL_BLAS=ON -DGMX_EXTERNAL_LAPACK=ON -DGMX_BLAS_USER= -DGMX_LAPACK_USER=`. Se não for o seu caso, apagar essas tags. Atenção ao `-DHIPSYCL_TARGETS='hip:gfxABC'`, substitua com os seus valores.

Agora é o momento de compilar, checar e instalar:

```
sudo make -j16 && sudo make check -j16
```
```
sudo make install -j16
```

Para carregar a biblioteca e invocar o Gromacs:

```
source /home/patrick/gromacs20250/bin/GMXRC
```
```
gmx -version
```

>[!WARNING]
>
>Durante `sudo make check -j16` ocorreram erros por TIMEOUT. Prossegui e testei uma dinâmica simples e não houve nenhum problema. Aparentemente, mais usuários do Gromacs 2024 enfrentam esses problemas e com `-DGMX_TEST_TIMEOUT_FACTOR=2` pode dar mais tempo para o teste.

>[!TIP]
>
>Você poderá editar o arquivo `/home/patrick/.bashrc` e adicionar o código `source /home/patrick/gromacs20250/bin/GMXRC`. Assim, toda vez que abrir o terminal já irá carregar o Gromacs.

🧪🧬⚗️ *Boas simulações moleculares!*

---
## 📜 Citação

- FAUSTINO, P. A. S. Tutorials: Compilando Gromacs 2025.x com ROCm e AdaptiveCpp/SyCL no Ubuntu 22.04, 2024. README. Disponível em: <[https://github.com/patrickallanfaustino/tutorials/blob/main/rocm-adaptivecpp-gromacs.md](https://github.com/patrickallanfaustino/tutorials/blob/main/rocm-adaptivecpp-gromacs-ptbr.md)>. Acesso em: [dia] de [mês] de [ano].

---
## 📝 Licença

Esse projeto está sob licença. Veja o arquivo [LICENÇA](LICENSE.md) para mais detalhes.
