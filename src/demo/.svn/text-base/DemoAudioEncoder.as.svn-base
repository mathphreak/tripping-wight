package demo
{
	import tonfall.core.samplingRate;
	import tonfall.format.wav.WAV16BitStereo44Khz;
	import tonfall.format.wav.WAVEncoder;

	import flash.display.Sprite;
	import flash.net.FileReference;
	import flash.utils.ByteArray;
	import flash.utils.getQualifiedClassName;

	/**
	 * Showcasing different audio encoders
	 * 
	 * Renders 100 cycles of 220Hz Sinewave
	 * 
	 * @author Andre Michelle
	 */
	public final class DemoAudioEncoder extends Sprite
	{
		private static const BUFFER_SIZE : int = 2048;
		
		private const buffer: ByteArray = new ByteArray();
		
		private const fileRef : FileReference = new FileReference();

//		private const encoder: WavEncoder = new WavEncoder( WAV8BitMono44Khz.INSTANCE );
//		private const encoder: WavEncoder = new WavEncoder( WAV8BitStereo44Khz.INSTANCE );
//		private const encoder: WavEncoder = new WavEncoder( WAV16BitMono44Khz.INSTANCE );
		private const encoder: WAVEncoder = new WAVEncoder( WAV16BitStereo44Khz.INSTANCE );
//		private const encoder: WavEncoder = new WavEncoder( WAV24BitMono44Khz.INSTANCE );
//		private const encoder: WavEncoder = new WavEncoder( WAV24BitStereo44Khz.INSTANCE );
//		private const encoder: WavEncoder = new WavEncoder( WAV32BitMono44Khz.INSTANCE );
//		private const encoder: WavEncoder = new WavEncoder( WAV32BitStereo44Khz.INSTANCE );

//		private const encoder: AiffEncoder = new AiffEncoder( AIFF8BitStereo44Khz.INSTANCE );
//		private const encoder: AiffEncoder = new AiffEncoder( AIFF16BitStereo44Khz.INSTANCE );
//		private const encoder: AiffEncoder = new AiffEncoder( AIFF24BitStereo44Khz.INSTANCE ); // TODO CREATES ARTIFACT
//		private const encoder: AiffEncoder = new AiffEncoder( AIFF32BitStereo44Khz.INSTANCE );

		public function DemoAudioEncoder()
		{
			buffer.length = BUFFER_SIZE << 3;
			
			create();
		}

		private function create(): void
		{
			const frequency: Number = 220.0;
			
			const numCircles: int = 100;
			
			var phase: Number = 0.0;
			
			var numSamples: int = samplingRate / frequency * numCircles;
			
			while( numSamples )
			{
				var write: int = Math.min( BUFFER_SIZE, numSamples );
				
				buffer.position = 0;
				
				for( var i: int = 0 ; i < write ; ++i )
				{
					var amplitude: Number = Math.sin( phase * 2.0 * Math.PI );
					
					buffer.writeFloat( amplitude );
					buffer.writeFloat( amplitude );
					
					phase += frequency / samplingRate;
					phase -= Math.floor( phase );
				}
				
				buffer.position = 0;
				
				encoder.write32BitStereo44KHz( buffer, write );
				
				numSamples -= write;
			}
			
			const name: String = getQualifiedClassName( encoder.strategy );
			
			fileRef.save( encoder.bytes, name.substr( name.indexOf( '::' ) + 2 ) + encoder.fileExt );
		}
	}
}