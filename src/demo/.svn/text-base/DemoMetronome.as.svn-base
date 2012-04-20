package demo
{
	import tonfall.display.AbstractApplication;
	import tonfall.prefab.metronome.MetronomeGenerator;
	import tonfall.prefab.metronome.MetronomeSequencer;

	/**
	 * Demoes simple musical event processing
	 * 
	 * @author Andre Michelle
	 */
	[SWF(width='640',height='480',backgroundColor='0x1b1b1b',frameRate='32',scriptTimeLimit='255')]
	public final class DemoMetronome extends AbstractApplication
	{
		public function DemoMetronome()
		{
			const sequencer: MetronomeSequencer = new MetronomeSequencer();
			const generator: MetronomeGenerator = new MetronomeGenerator();

			sequencer.timeEventTarget = generator;

			engine.processors.push( sequencer );
			engine.processors.push( generator );

			engine.input = generator.output;
			
			showSpectrum = true;
		}
	}
}