-- 1. Create the PUBLIC 'assets' bucket
-- We set public = true so files can be accessed without a token (via the /public/ endpoint)
INSERT INTO storage.buckets (id, name, public) 
VALUES ('assets', 'assets', true);

-- 2. Create the PRIVATE 'vault' bucket
-- We set public = false so files require a Bearer token (which our Nginx proxy handles)
INSERT INTO storage.buckets (id, name, public) 
VALUES ('vault', 'vault', false);

-- Allow the whole world to view files in 'assets'
CREATE POLICY "Public Assets are viewable by everyone" 
ON storage.objects FOR SELECT 
USING ( bucket_id = 'assets' );

-- Allow logged-in users to upload to 'assets'
CREATE POLICY "Authenticated users can upload assets" 
ON storage.objects FOR INSERT 
WITH CHECK ( 
  bucket_id = 'assets' 
  AND auth.role() = 'authenticated' 
);

-- Allow users full access (Select, Insert, Update, Delete) ONLY to their own folder
CREATE POLICY "Users can manage their own vault files" 
ON storage.objects FOR ALL 
USING (
    bucket_id = 'vault' 
    AND auth.uid()::text = (storage.foldername(name))[1]
)
WITH CHECK (
    bucket_id = 'vault' 
    AND auth.uid()::text = (storage.foldername(name))[1]
);