


{:.no_toc}
* toc
{:toc}



# Abstract
Denoising diffusion probabilistic models (DDPMs) have recently achieved leading performances in many generative tasks. However,
the inherited iterative sampling process costs hindered their applications to text-to-speech synthesis deployment. Through the
preliminary study on diffusion model parameterization, we find that previous gradient-based TTS models require hundreds or thousands of iterations to guarantee high sample quality, which poses a challenge for accelerating sampling. In this work, we propose ProDiff, on progressive fast diffusion model for high-quality textto-speech. Unlike estimating the gradient for data density, ProDiff parameterizes the denoising model by directly predicting clean data to avoid distinct quality degradation when reducing sampling iterations. To tackle the model convergence challenge with reduced iterations, ProDiff reduces the data variance in the target site via knowledge distillation. Specifically, the denoising model uses the
generated mel-spectrogram from an N-step DDIM teacher as the
training target and distills the behavior into a new model with N/2
steps. As such, it allows the TTS model to make sharp predictions
and further reduces the sampling time by orders of magnitude. Our
evaluation demonstrates that ProDiff needs only 2 iterations to synthesize high-fidelity mel-spectrograms, while it maintains sample
quality and diversity competitive with state-of-the-art models using
hundreds of steps. ProDiff enables a sampling speed of 24x faster
than real-time on a single NVIDIA 2080Ti GPU, making diffusion
models practically applicable to text-to-speech synthesis deployment for the first time. Our extensive ablation studies demonstrate
that each design in ProDiff is effective, and we further show that
ProDiff can be easily extended to the multi-speaker setting.
Audio samples are available at <a href="https://prodiff.github.io/"><i>https://prodiff.github.io/</i></a>.


# Progressive Fast Diffusion Model for High-Quality
## Preliminary Analyses


<ruby>Reference Text: This type was introduced into England by Wynkyn de Worde, Caxton's successor</ruby>
<table>
    <thead>
    <th style="text-align: center">Method</th>
    <th style="text-align: center">Recording</th>
    <th style="text-align: center">128 iter</th>
    <th style="text-align: center">64 iter</th>
    <th style="text-align: center">32 iter</th>
    <th style="text-align: center">16 iter</th>
    <th style="text-align: center">8 iter</th>
    <th style="text-align: center">4 iter</th>
    <th style="text-align: center">2 iter</th>
    <!-- <th style="text-align: center">FB MelGAN</th>
    <th style="text-align: center">NSF</th>
    <th style="text-align: center">SingGAN</th> -->
    </thead>
    <tbody>
        <tr>
            <th>Gradient-Based</th>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/GT/[000000][LJ001-0069][G].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-128/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-64/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-32/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-16/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-8/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-4/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-2/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
        </tr>
    </tbody>
    <tbody>
        <tr>
        <th>Generator-Based</th>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/GT/[000000][LJ001-0069][G].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-128/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-64/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-32/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-16/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-8/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-4/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-2/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
        </tr>
    </tbody>
    
</table>

<ruby>Reference Text: the ends of many of the letters such as the t and e are hooked up in a vulgar and meaningless way</ruby>
<table>
    <thead>
    <th style="text-align: center">Method</th>
    <th style="text-align: center">Recording</th>
    <th style="text-align: center">128 iter</th>
    <th style="text-align: center">64 iter</th>
    <th style="text-align: center">32 iter</th>
    <th style="text-align: center">16 iter</th>
    <th style="text-align: center">8 iter</th>
    <th style="text-align: center">4 iter</th>
    <th style="text-align: center">2 iter</th>
    <!-- <th style="text-align: center">FB MelGAN</th>
    <th style="text-align: center">NSF</th>
    <th style="text-align: center">SingGAN</th> -->
    </thead>
    <tbody>
        <tr>
            <th>Gradient-Based</th>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/GT/[000004][LJ001-0111][G].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-128/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-64/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-32/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-16/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-8/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-4/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-2/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
        </tr>
    </tbody>
    <tbody>
        <tr>
            <th>Generator-Based</th>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/GT/[000004][LJ001-0111][G].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-128/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-64/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-32/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-16/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-8/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-4/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-2/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
        </tr>
    </tbody>
    
</table>

Reference Text: The essential point to be remembered is that the ornament, whatever it is, whether picture or pattern-work, should form part of the page
<table>
    <thead>
    <th style="text-align: center">Method</th>
    <th style="text-align: center">Recording</th>
    <th style="text-align: center">128 iter</th>
    <th style="text-align: center">64 iter</th>
    <th style="text-align: center">32 iter</th>
    <th style="text-align: center">16 iter</th>
    <th style="text-align: center">8 iter</th>
    <th style="text-align: center">4 iter</th>
    <th style="text-align: center">2 iter</th>
    <!-- <th style="text-align: center">FB MelGAN</th>
    <th style="text-align: center">NSF</th>
    <th style="text-align: center">SingGAN</th> -->
    </thead>
    <tbody>
        <tr>
            <th>Gradient-Based</th>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/GT/[000005][LJ001-0173][G].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-128/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-64/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-32/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-16/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-8/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-4/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-2/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
        </tr>
    </tbody>
    <tbody>
        <tr>
            <th>Generator-Based</th>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/GT/[000005][LJ001-0173][G].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-128/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-64/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-32/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-16/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-8/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-4/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-2/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
        </tr>
    </tbody>
    
</table>

<ruby>Reference Text: The prison population fluctuated a great deal</ruby>
<table>
    <thead>
    <th style="text-align: center">Method</th>
    <th style="text-align: center">Recording</th>
    <th style="text-align: center">128 iter</th>
    <th style="text-align: center">64 iter</th>
    <th style="text-align: center">32 iter</th>
    <th style="text-align: center">16 iter</th>
    <th style="text-align: center">8 iter</th>
    <th style="text-align: center">4 iter</th>
    <th style="text-align: center">2 iter</th>
    <!-- <th style="text-align: center">FB MelGAN</th>
    <th style="text-align: center">NSF</th>
    <th style="text-align: center">SingGAN</th> -->
    </thead>
    <tbody>
        <tr>
            <th>Gradient-Based</th>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/GT/[000006][LJ002-0005][G].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-128/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-64/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-32/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-16/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-8/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-4/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-2/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
        </tr>
    </tbody>
    <tbody>
        <tr>
            <th>Generator-Based</th>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/GT/[000006][LJ002-0005][G].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-128/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-64/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-32/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-16/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-8/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-4/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-2/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
        </tr>
    </tbody>
    
</table>

## Performance

<ruby>Reference/Target Text: This type was introduced into England by Wynkyn de Worde, Caxton's successor</ruby>
<table>
	<thead>
		<tr>
			<th style="text-align: center">Recording</th>
            <th style="text-align: center">Tacotron2</th>
            <th style="text-align: center">FastSpeech 2</th>
            <th style="text-align: center">GANSpeech</th>
            <th style="text-align: center">Glow-TTS</th>
            <th style="text-align: center">Grad-TTS</th>
            <th style="text-align: center">DiffSpeech</th>
            <th style="text-align: center">ProDiff Teacher</th>
            <th style="text-align: center">ProDiff</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/GT/[000000][LJ001-0069][G].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/Tacotron 2/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/FastSpeech 2/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/GanSpeech/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/Glow-TTS/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/Grad-TTS/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/DiffSpeech/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/ProDiff Teacher/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/ProDiff/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
		</tr>
	</tbody>
</table>

<ruby>Reference/Target Text: the ends of many of the letters such as the t and e are hooked up in a vulgar and meaningless way</ruby>
<table>
	<thead>
		<tr>
			<th style="text-align: center">Recording</th>
            <th style="text-align: center">Tacotron2</th>
            <th style="text-align: center">FastSpeech 2</th>
            <th style="text-align: center">GANSpeech</th>
            <th style="text-align: center">Glow-TTS</th>
            <th style="text-align: center">Grad-TTS</th>
            <th style="text-align: center">DiffSpeech</th>
            <th style="text-align: center">ProDiff Teacher</th>
            <th style="text-align: center">ProDiff</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/GT/[000004][LJ001-0111][G].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/Tacotron 2/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/FastSpeech 2/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/GanSpeech/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/Glow-TTS/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/Grad-TTS/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/DiffSpeech/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/ProDiff Teacher/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/ProDiff/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
		</tr>
	</tbody>
</table>

Reference/Target Text: The essential point to be remembered is that the ornament, whatever it is, whether picture or pattern-work, should form part of the page
<table>
	<thead>
		<tr>
			<th style="text-align: center">Recording</th>
            <th style="text-align: center">Tacotron2</th>
            <th style="text-align: center">FastSpeech 2</th>
            <th style="text-align: center">GANSpeech</th>
            <th style="text-align: center">Glow-TTS</th>
            <th style="text-align: center">Grad-TTS</th>
            <th style="text-align: center">DiffSpeech</th>
            <th style="text-align: center">ProDiff Teacher</th>
            <th style="text-align: center">ProDiff</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/GT/[000005][LJ001-0173][G].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/DiffSpeech/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/FastSpeech 2/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/GanSpeech/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/Glow-TTS/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/Grad-TTS/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/DiffSpeech/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/ProDiff Teacher/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/ProDiff/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
		</tr>
	</tbody>
</table>

<ruby>Reference/Target Text: The prison population fluctuated a great deal</ruby>
<table>
	<thead>
		<tr>
			<th style="text-align: center">Recording</th>
            <th style="text-align: center">Tacotron2</th>
            <th style="text-align: center">FastSpeech 2</th>
            <th style="text-align: center">GANSpeech</th>
            <th style="text-align: center">Glow-TTS</th>
            <th style="text-align: center">Grad-TTS</th>
            <th style="text-align: center">DiffSpeech</th>
            <th style="text-align: center">ProDiff Teacher</th>
            <th style="text-align: center">ProDiff</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/GT/[000006][LJ002-0005][G].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/Tacotron 2/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/FastSpeech 2/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/GanSpeech/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/Glow-TTS/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/Grad-TTS/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/DiffSpeech/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/ProDiff Teacher/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table2/ProDiff/[000006][LJ002-0005]][P].wav" type="audio/wav"></audio></td>
		</tr>
	</tbody>
</table>


## Ablation 


<ruby>Reference/Target Text: This type was introduced into England by Wynkyn de Worde, Caxton's successor</ruby>
<table>
	<thead>
		<tr>
			<th style="text-align: center">Recording</th>
            <th style="text-align: center">w/o GP</th>
            <th style="text-align: center">w/o KD</th>
            <th style="text-align: center">Teacher(T=16)</th>
            <th style="text-align: center">Teacher(T=8)</th>
            <th style="text-align: center">ProDiff</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/GT/[000000][LJ001-0069][G].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-2/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-2/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/ProDiff-16/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/ProDiff-8/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/ProDiff-2/[000000][LJ001-0069][P].wav" type="audio/wav"></audio></td>
		</tr>
	</tbody>
</table>

<ruby>Reference/Target Text: the ends of many of the letters such as the t and e are hooked up in a vulgar and meaningless way</ruby>
<table>
	<thead>
		<tr>
			<th style="text-align: center">Recording</th>
            <th style="text-align: center">w/o GP</th>
            <th style="text-align: center">w/o KD</th>
            <th style="text-align: center">Teacher(T=16)</th>
            <th style="text-align: center">Teacher(T=8)</th>
            <th style="text-align: center">ProDiff</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/GT/[000004][LJ001-0111][G].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-2/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-2/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/ProDiff-16/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/ProDiff-8/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/ProDiff-2/[000004][LJ001-0111][P].wav" type="audio/wav"></audio></td>
		</tr>
	</tbody>
</table>

Reference/Target Text: The essential point to be remembered is that the ornament, whatever it is, whether picture or pattern-work, should form part of the page
<table>
	<thead>
		<tr>
			<th style="text-align: center">Recording</th>
            <th style="text-align: center">w/o GP</th>
            <th style="text-align: center">w/o KD</th>
            <th style="text-align: center">Teacher(T=16)</th>
            <th style="text-align: center">Teacher(T=8)</th>
            <th style="text-align: center">ProDiff</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/GT/[000005][LJ001-0173][G].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-2/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-2/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/ProDiff-16/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/ProDiff-8/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/ProDiff-2/[000005][LJ001-0173][P].wav" type="audio/wav"></audio></td>
		</tr>
	</tbody>
</table>

<ruby>Reference/Target Text: The prison population fluctuated a great deal</ruby>
<table>
	<thead>
		<tr>
			<th style="text-align: center">Recording</th>
            <th style="text-align: center">w/o GP</th>
            <th style="text-align: center">w/o KD</th>
            <th style="text-align: center">Teacher(T=16)</th>
            <th style="text-align: center">Teacher(T=8)</th>
            <th style="text-align: center">ProDiff</th>
		</tr>
	</thead>
	<tbody>
		<tr>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/GT/[000006][LJ002-0005][G].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Score-2/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table1/Generator-2/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/ProDiff-16/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/ProDiff-8/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
            <td style="text-align: center"><audio controls style="width: 150px;"><source src="wav_for_demo/table3/ProDiff-2/[000006][LJ002-0005][P].wav" type="audio/wav"></audio></td>
		</tr>
	</tbody>
</table>


